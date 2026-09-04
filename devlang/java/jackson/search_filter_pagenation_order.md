
## FE json




##
```proto
// ===========================
// 보고서 계획 목록 조회
// ==============================

message ReportPlanListRequestV1 {
  ReportPlanFilterV1 filter = 1;
  ReportPlanPaginationV1 pagination = 2;
  ReportPlanSortV1 sort = 3;
}

message ReportPlanFilterV1 {
  oneof condition {
    ReportPlanFilterAndV1 and = 1;
    ReportPlanFilterSearchStringV1 search_string = 2;
    ReportPlanFilterStatusV1 status = 3;
    ReportPlanFilterReportCategoryV1 report_category = 4;
    ReportPlanFilterScheduleFrequencyV1 schedule_frequency = 5;
  }
}

message ReportPlanFilterAndV1 {
  repeated ReportPlanFilterV1 targets = 1;
}

message ReportPlanFilterSearchStringV1 {
  string value = 1;
}

message ReportPlanFilterStatusV1 {
  repeated string values = 1;
}

message ReportPlanFilterReportCategoryV1 {
  repeated string values = 1;
}

message ReportPlanFilterScheduleFrequencyV1 {
  repeated string values = 1;
}

message ReportPlanListResponseV1 {
  int64 total_count = 1;
  repeated ReportPlanV1 plans = 2;
}
```