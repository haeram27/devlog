# Spring에서 MongoDB query하기

```java
import java.time.Instant;
import java.time.LocalDateTime;
import java.time.OffsetDateTime;
import java.time.ZoneId;
import java.time.format.DateTimeFormatter;
import java.time.format.DateTimeParseException;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;
import java.util.Map;
import java.util.Optional;
import org.bson.Document;
import org.bson.json.Converter;
import org.bson.json.JsonMode;
import org.bson.json.JsonWriterSettings;
import org.bson.json.StrictJsonWriter;
import org.bson.types.ObjectId;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.Sort;
import org.springframework.data.domain.Sort.Direction;
import org.springframework.data.domain.Sort.Order;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.data.mongodb.core.query.Criteria;
import org.springframework.data.mongodb.core.query.Query;
import org.springframework.stereotype.Service;
import org.springframework.util.CollectionUtils;
import org.springframework.util.StringUtils;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;

@Service
@Slf4j
@RequiredArgsConstructor
public class MongoCommonFindService {
    private final MongoTemplate mongoTemplate;
    private final ObjectMapper objectMapper;

    private static final String MONGO_OBJECT_ID_KEY = "_id";
    private static final String TIMEZONE_ID_UTC = "UTC";

    private static final int MIN_PAGE_NUMBER = 1;
    private static final int MIN_PAGE_SIZE = 1;
    private static final int DEFAULT_PAGE_SIZE = 100000;

    @Data
    class MongoObjectIdCriteria {
        private String timezoneId;
        private String startDate;  // '2011-12-03T10:15:30' DateTimeFormatter.ISO_LOCAL_DATE_TIME
        private String endDate;    // '2011-12-03T10:15:30' DateTimeFormatter.ISO_LOCAL_DATE_TIME
    }

    @Data
    class MongoDateCriteria {
        private String timezoneId;
        private String startDate; // '2011-12-03T10:15:30.123' DateTimeFormatter.ISO_LOCAL_DATE_TIME
        private String endDate; // '2011-12-03T10:15:30.123' DateTimeFormatter.ISO_LOCAL_DATE_TIME
        private String key;
    }

    @Data
    class MongoCommonFindReq {
        private String collection;
        private String respTimezoneId; // response timezone id ex: Asia/Seoul
        private MongoObjectIdCriteria idCriteria;
        private List<MongoDateCriteria> dateCriterias;
        private Map<String, Object> equalCriterias;
        private Map<String, List<Object>> inCriterias;
        private Map<String, String> regexCriterias;
        private List<String> projectKeys;
        private Map<String, String> orders;
        private Integer pageSize;
        private Integer pageNumber;
        private Integer revision;
    }

    private class ObjectIdConverter implements Converter<ObjectId> {
        @Override
        public void convert(ObjectId value, StrictJsonWriter writer) {
            writer.writeString(value.toHexString());
        }
    }

    private class DateTimeConverter implements Converter<Long> {
        private String zoneId;

        DateTimeConverter(String zoneId) {
            this.zoneId = zoneId;
        }

        @Override
        public void convert(Long value, StrictJsonWriter writer) {
            writer.writeString(
                DateTimeFormatter.ISO_OFFSET_DATE_TIME.format(
                    OffsetDateTime.ofInstant(Instant.ofEpochMilli(value), ZoneId.of(zoneId))));
        }
    }

    public List<Map<String, Object>> find(MongoCommonFindReq request) throws RuntimeException {
        Query query = new Query();

        String collection = StringUtils.hasText(request.getCollection()) ? request.getCollection() : "";
        if (!StringUtils.hasText(collection)) {
            throw new RuntimeException("invalid collection name");
        }

        String respTimezoneId = StringUtils.hasText(request.getRespTimezoneId()) ? request.getRespTimezoneId() : TIMEZONE_ID_UTC;

        if (request.getIdCriteria() != null) {
            var idCreteria = request.getIdCriteria();
            String tzId = StringUtils.hasText(idCreteria.getTimezoneId()) ? idCreteria.getTimezoneId() : TIMEZONE_ID_UTC;
            String idStartDate = StringUtils.hasText(idCreteria.getStartDate()) ? idCreteria.getStartDate() : "";
            String idEndDate = StringUtils.hasText(idCreteria.getEndDate()) ? idCreteria.getEndDate() : "";
            if (StringUtils.hasText(idStartDate) && StringUtils.hasText(idEndDate)) {
                try {
                    var fromLocalTime = LocalDateTime.parse(idStartDate, DateTimeFormatter.ISO_LOCAL_DATE_TIME);
                    var toLocalTime = LocalDateTime.parse(idEndDate, DateTimeFormatter.ISO_LOCAL_DATE_TIME);
                    var fromInstant = fromLocalTime.toInstant(ZoneId.of(tzId).getRules().getOffset(fromLocalTime));
                    var toInstant = toLocalTime.toInstant(ZoneId.of(tzId).getRules().getOffset(toLocalTime));
                    var fromId = new ObjectId(Date.from(fromInstant));
                    var toId = new ObjectId(Date.from(toInstant));
                    query.addCriteria(Criteria.where(MONGO_OBJECT_ID_KEY).gte(fromId).lt(toId));
                } catch (DateTimeParseException e) {
                    throw new RuntimeException(e);
                }
            }
        }

        if (request.getDateCriterias() != null && !request.getDateCriterias().isEmpty()) {
            request.getDateCriterias().forEach(cri -> {
                String tzId = StringUtils.hasText(cri.getTimezoneId()) ? cri.getTimezoneId() : TIMEZONE_ID_UTC;
                String startDate = StringUtils.hasText(cri.getStartDate()) ? cri.getStartDate() : "";
                String endDate = StringUtils.hasText(cri.getEndDate()) ? cri.getEndDate() : "";
                if (StringUtils.hasText(startDate) && StringUtils.hasText(endDate)) {
                    try {
                        var fromLocalTime = LocalDateTime.parse(startDate, DateTimeFormatter.ISO_LOCAL_DATE_TIME);
                        var toLocalTime = LocalDateTime.parse(endDate, DateTimeFormatter.ISO_LOCAL_DATE_TIME);
                        var fromInstant = fromLocalTime
                                .toInstant(ZoneId.of(tzId).getRules().getOffset(fromLocalTime));
                        var toInstant = toLocalTime.toInstant(ZoneId.of(tzId).getRules().getOffset(toLocalTime));
                        var from = Date.from(fromInstant);
                        var to = Date.from(toInstant);
                        log.info("from : {}", from);

                        query.addCriteria(Criteria.where(cri.getKey()).gte(from).lt(to));
                    } catch (DateTimeParseException e) {
                        throw new RuntimeException(e);
                    }
                }
            });
        }

        var equalCriterias = request.getEqualCriterias();
        if (! CollectionUtils.isEmpty(equalCriterias)) {
            equalCriterias.forEach((k, v) -> {
                query.addCriteria(Criteria.where(k).is(v));
            });
        }

        var inCriterias = request.getEqualCriterias();
        if (! CollectionUtils.isEmpty(inCriterias)) {
            inCriterias.forEach((k, v) -> {
                if (v instanceof List) query.addCriteria(Criteria.where(k).in(v));
            });
        }

        var regexCriterias = request.getRegexCriterias();
        if (! CollectionUtils.isEmpty(regexCriterias)) {
            regexCriterias.forEach((k, v) -> {
                query.addCriteria(Criteria.where(k).regex(v));
            });
        }

        var projectKeys = Optional.ofNullable(request.getProjectKeys());
        if (projectKeys.isPresent() && projectKeys.get().size() > 0) {
            projectKeys.get().forEach((key) -> {
                query.fields().include(key);
            });
        }

        /* Sorting */
        var orders = new ArrayList<Order>();
        var reqOrders = request.getOrders();
        if (CollectionUtils.isEmpty(reqOrders)) {
            orders.add(new Order(Direction.DESC, MONGO_OBJECT_ID_KEY));
        } else {
            reqOrders.forEach((k, v) -> {
                if ("asc".equalsIgnoreCase((String) v)) {
                    orders.add(new Order(Direction.ASC, k));
                } else if ("desc".equalsIgnoreCase((String) v)) {
                    orders.add(new Order(Direction.DESC, k));
                };
            });
        }
        query.with(Sort.by(orders));

        /* Pagenation */
        int pageNumber = Optional.ofNullable(request.getPageNumber()).orElse(0); // zero-based page number, must not be negative.
        int pageSize = Optional.ofNullable(request.getPageSize()).orElse(0); // number of docs per a page to be returned, must be greater than 0.

        if (pageNumber < MIN_PAGE_NUMBER)
            pageNumber = MIN_PAGE_NUMBER;
        if (pageSize < MIN_PAGE_SIZE)
            pageSize = DEFAULT_PAGE_SIZE;

        Pageable pageable = PageRequest.of(pageNumber - MIN_PAGE_NUMBER, pageSize);
        query.with(pageable);

        /* Get Results */
        List<Document> results = mongoTemplate.find(query, Document.class, collection);
        log.debug("nosql find({}): results.size = {}", collection, results.size());

        /* adjust json value as favorite format */
        JsonWriterSettings writerSettings = JsonWriterSettings.builder()
                .outputMode(JsonMode.RELAXED)
                // .objectIdConverter(new ObjectIdConverter())
                // .dateTimeConverter(new DateTimeConverter(respTimezoneId))
                .build();

        var mappedResults = new ArrayList<Map<String, Object>>();
        for (var e : results) {
            try {
                mappedResults.add(objectMapper.readValue(
                    e.toJson(writerSettings),
                    new TypeReference<Map<String, Object>>() {}));
            } catch (JsonProcessingException ex) {
                throw new RuntimeException(ex);
            }
        }

        return mappedResults;
    }
```