
# mockito

## annotations

1. `@Mock`
   - Mockito가 **가짜 객체(목 객체)** 를 생성합니다.
   - 외부 의존성의 동작을 `when(...).thenReturn(...)`으로 지정할 때 사용합니다.
   - 테스트 대상이 아니라, **테스트 대상이 의존하는 객체**에 붙입니다.

2. `@InjectMocks`
   - Mockito가 **테스트 대상 객체**를 만들고,
   - 같은 테스트 클래스 안의 `@Mock` 객체들을 생성자/필드/세터로 주입해 줍니다.
   - 즉, “이 클래스는 진짜로 테스트하되, 내부 의존성만 Mock으로 바꿔서 넣어라”는 의미입니다.

3. `@Spy`
   - 실제 객체를 감싸는 Spy 객체를 만듭니다.
   - 기본 동작은 실제 메서드를 호출하고, 필요한 메서드만 선택적으로 stub 할 때 사용합니다.
   - 부분 mock이므로 과도하게 사용하면 테스트 의도가 흐려질 수 있습니다.

4. `@Captor`
   - `ArgumentCaptor`를 간결하게 선언할 때 사용합니다.
   - mock 메서드 호출 시 전달된 인자 값을 잡아서(assertion) 검증할 수 있습니다.

5. `@ExtendWith(MockitoExtension.class)` (JUnit 5)
   - Mockito annotation(`@Mock`, `@InjectMocks`, `@Spy`, `@Captor`) 초기화를 자동으로 처리합니다.
   - JUnit 5에서 Mockito를 사용할 때 사실상 기본 설정입니다.

6. `@MockitoSettings(strictness = Strictness.STRICT_STUBS)`
   - 사용되지 않는 stubbing 등을 테스트 단계에서 엄격하게 검출합니다.
   - 불필요한 mock 설정을 줄여 테스트 품질을 높일 수 있습니다.

7. `@Nested` + `@DisplayName` (권장)
   - Mockito 전용은 아니지만, 테스트 시나리오를 계층적으로 정리할 때 매우 유용합니다.
   - 읽기 쉬운 테스트 문서화에 도움이 됩니다.


## methods

아래는 Mockito에서 **가장 자주 쓰고 반드시 알아야 하는 메서드**입니다.

1. Stubbing
   - `when(mock.method(...)).thenReturn(value)`
       - 의미: 테스트 중 해당 mock 메서드가 호출되면 **지정한 값을 반환**하게 만듭니다.
       - 용도: 외부 의존성(DB, API, 메시지 브로커) 결과를 고정해 테스트를 예측 가능하게 만듭니다.
       - `when(...)`이 받는 인자는 **값(value)** 이 아니라 **mock(또는 spy)의 메서드 호출식(method call expression)** 입니다.
       - 예: `when(userRepository.findById(1L)).thenReturn(Optional.of(user))`
       - 제네릭 시그니처 관점에서는 `when(T methodCall)` 형태이며, `T`는 해당 메서드의 반환 타입입니다.
   - `when(...).thenThrow(exception)`
       - 의미: 해당 호출 시 **예외를 던지도록** 설정합니다.
       - 용도: 네트워크 오류, 중복 데이터, 권한 오류 같은 실패 경로를 검증합니다.
   - `when(...).thenAnswer(answer)`
       - 의미: 호출 시점의 인자를 읽어 **동적으로 반환값을 계산**합니다.
       - 용도: 입력값마다 다른 응답이 필요한 경우(예: save 시 전달 객체 그대로 반환)에 사용합니다.
   - `doReturn(value).when(spy).method(...)`
       - 의미: spy에서 실제 메서드 실행 없이 반환값만 지정합니다.
       - 용도: spy 대상 메서드가 무겁거나 부작용이 있을 때 안전하게 stub 합니다.
   - `doThrow(exception).when(mock).voidMethod(...)`
       - 의미: void 메서드 호출 시 예외를 던지게 합니다.
       - 용도: 알림 전송, 파일 삭제 등 void 작업 실패 처리 테스트에 사용합니다.
   - `doNothing().when(mock).voidMethod(...)`
       - 의미: void 메서드 호출을 무해하게 통과시킵니다.
       - 용도: 실제 실행 시 부작용이 있는 void 메서드를 테스트에서 비활성화할 때 사용합니다.

    주의:
    - `when(...)`은 반환값이 있는 메서드 stubbing에 주로 사용합니다.
    - void 메서드는 `when(...)` 대신 `doThrow(...).when(...)`, `doNothing().when(...)`, `doAnswer(...).when(...)` 패턴을 사용합니다.

2. Verification
   - `verify(mock).method(...)`
       - 의미: 해당 메서드가 **적어도 1번 호출되었는지** 검증합니다.
       - 용도: "실행 결과"뿐 아니라 "협력 객체와의 상호작용"까지 확인합니다.
   - `verify(mock, times(n)).method(...)`
       - 의미: 호출 횟수가 정확히 `n`번인지 검증합니다.
       - 용도: 중복 호출 버그나 누락 호출 버그를 잡습니다.
   - `verify(mock, never()).method(...)`
       - 의미: 해당 메서드가 한 번도 호출되지 않았음을 검증합니다.
       - 용도: 실패 시나리오에서 금지된 부작용(save, publish 등)이 없는지 확인합니다.
   - `verifyNoInteractions(mock)`
       - 의미: 해당 mock 객체에 어떤 호출도 없었는지 검증합니다.
       - 용도: 특정 분기에서 완전히 건드리면 안 되는 의존성을 보호할 때 사용합니다.
   - `verifyNoMoreInteractions(mock)`
       - 의미: 이미 검증한 호출 외에는 추가 상호작용이 없음을 검증합니다.
       - 용도: 불필요한 호출(숨은 부작용)을 엄격하게 통제할 때 사용합니다.
   - `InOrder inOrder = inOrder(mock1, mock2); inOrder.verify(...)`
       - 의미: 여러 mock 메서드의 호출 순서를 검증합니다.
       - 용도: 결제 후 저장, 잠금 후 갱신처럼 순서가 중요한 비즈니스 로직 검증에 사용합니다.

3. Argument Matching
   - `any()`, `anyString()`, `anyLong()`
       - 의미: 해당 타입의 임의 값을 허용하는 matcher 입니다.
       - 용도: 특정 값보다 "호출 자체"에 집중하고 싶을 때 사용합니다.
   - `eq(value)`
       - 의미: 전달 인자가 특정 값과 정확히 같은지 매칭합니다.
       - 용도: 일부 인자는 정확 값, 일부 인자는 any로 유연하게 매칭할 때 사용합니다.
   - `argThat(predicate)`
       - 의미: 람다/조건식으로 커스텀 매칭 규칙을 정의합니다.
       - 용도: 객체 내부 필드 조건(예: amount > 0, status == READY) 검증에 사용합니다.

주의:
- matcher를 쓸 때는 같은 호출 인자에서 raw value와 섞지 않는 것이 안전합니다.
- 예: `method(anyString(), eq(10))`는 OK, `method(anyString(), 10)`은 지양

4. Argument Capturing
   - `ArgumentCaptor<T> captor`
       - 의미: mock 호출 시 전달된 실제 인자를 나중에 꺼내 검증할 수 있게 잡아 둡니다.
       - 용도: 메서드 호출 여부를 넘어, 정확히 어떤 데이터가 전달됐는지 검증합니다.
   - `verify(mock).method(captor.capture())`
       - 의미: verify 단계에서 인자를 캡처합니다.
       - 용도: save/publish 호출의 payload 검증에 사용합니다.
   - `captor.getValue()` 또는 `captor.getAllValues()`로 전달값 검증
       - 의미: 마지막 1개 또는 전체 호출 인자를 조회합니다.
       - 용도: 단건/복수 호출 케이스의 전달 데이터 검증에 사용합니다.

5. 기타 유틸
   - `lenient().when(...)`
       - 의미: strict stubbing 검사에서 해당 stubbing만 예외 처리합니다.
       - 용도: 공통 설정 중 일부 테스트에서만 미사용될 수 있는 stubbing에 제한적으로 사용합니다.
   - `reset(mock)`
       - 의미: mock의 stubbing/interaction 기록을 초기화합니다.
       - 용도: 특별한 경우에만 사용하고, 일반적으로는 테스트를 분리하는 구조가 더 권장됩니다.

## examples

아래 예제는 JUnit 5 + Mockito 기준입니다.

### 예제 1) 서비스 단위 테스트 (성공/실패/검증)

```java
import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.ArgumentMatchers.*;
import static org.mockito.Mockito.*;

import java.util.Optional;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.ArgumentCaptor;
import org.mockito.Captor;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

@ExtendWith(MockitoExtension.class)
class UserServiceTest {

        @Mock
        private UserRepository userRepository;

        @Mock
        private PasswordEncoder passwordEncoder;

        @InjectMocks
        private UserService userService;

        @Captor
        private ArgumentCaptor<User> userCaptor;

        @Test
        @DisplayName("회원 가입 시 비밀번호 인코딩 후 저장한다")
        void register_success() {
                // given
                when(passwordEncoder.encode("plainPw")).thenReturn("encodedPw");
                when(userRepository.save(any(User.class))).thenAnswer(invocation -> invocation.getArgument(0));

                // when
                User saved = userService.register("alice", "plainPw");

                // then
                assertEquals("alice", saved.getUsername());
                assertEquals("encodedPw", saved.getPassword());

                verify(passwordEncoder, times(1)).encode("plainPw");
                verify(userRepository).save(userCaptor.capture());
                assertEquals("encodedPw", userCaptor.getValue().getPassword());
                verifyNoMoreInteractions(passwordEncoder, userRepository);
        }

        @Test
        @DisplayName("이미 존재하는 사용자면 예외를 던진다")
        void register_duplicateUser_throws() {
                // given
                when(userRepository.findByUsername("alice")).thenReturn(Optional.of(new User("alice", "x")));

                // when / then
                assertThrows(IllegalStateException.class,
                                () -> userService.registerIfNotExists("alice", "plainPw"));

                verify(userRepository).findByUsername("alice");
                verify(userRepository, never()).save(any());
        }
}
```

설명:
- `@InjectMocks`로 `UserService`를 실제 객체로 생성하고, 의존성은 `@Mock`으로 주입합니다.
- 성공 케이스는 stubbing + captor + 호출 검증을 함께 사용합니다.
- 실패 케이스는 `assertThrows`와 `never()`로 부작용이 없었는지 확인합니다.

### 예제 2) void 메서드 예외 처리 검증

```java
import static org.junit.jupiter.api.Assertions.assertThrows;
import static org.mockito.Mockito.doThrow;
import static org.mockito.Mockito.verify;

import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

@ExtendWith(MockitoExtension.class)
class NotificationServiceTest {

        @Mock
        private MessageSender sender;

        @InjectMocks
        private NotificationService notificationService;

        @Test
        void send_fails_when_sender_throws() {
                doThrow(new RuntimeException("network error"))
                                .when(sender)
                                .send("hello");

                assertThrows(RuntimeException.class, () -> notificationService.notify("hello"));
                verify(sender).send("hello");
        }
}
```

설명:
- void 메서드는 `when(...).thenThrow(...)` 대신 `doThrow(...).when(...)` 패턴을 사용합니다.

### 예제 3) 호출 순서 검증 (InOrder)

```java
import static org.mockito.Mockito.inOrder;
import static org.mockito.Mockito.verify;

import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InOrder;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

@ExtendWith(MockitoExtension.class)
class CheckoutServiceTest {

        @Mock
        private PaymentClient paymentClient;

        @Mock
        private OrderRepository orderRepository;

        @InjectMocks
        private CheckoutService checkoutService;

        @Test
        void checkout_calls_payment_then_save_order() {
                checkoutService.checkout(1000L);

                InOrder inOrder = inOrder(paymentClient, orderRepository);
                inOrder.verify(paymentClient).pay(1000L);
                inOrder.verify(orderRepository).saveCompletedOrder(1000L);
        }
}
```

설명:
- 결과값만 보는 테스트로는 놓치기 쉬운 "호출 순서"를 보장합니다.

## 정리

- annotation 핵심: `@Mock`, `@InjectMocks`, `@Captor`, `@Spy`, `@ExtendWith(MockitoExtension.class)`
- method 핵심: `when/thenReturn`, `doThrow`, `verify/times/never`, matcher(`any/eq`), `ArgumentCaptor`, `InOrder`
- 좋은 Mockito 테스트는
  - 테스트 대상은 실제 객체로 두고
  - 외부 의존성만 mock 처리하며
  - 반환값 + 부작용(호출/인자/순서)까지 함께 검증합니다.