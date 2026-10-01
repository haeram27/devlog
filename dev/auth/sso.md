# Single Sign On

- Single Sign On
  - Login once to multiple applciations

- LDAP
  - Protocol for accessing directory services to look up users and verify credentials.
- Active Directory
  - Microsoft's directory service for managing users and authenticating them across a domain.

- OpenID Connect
  - Authentication protocol built on OAuth 2.0 that provides user identity information.
- OAuth 2.0
  - Authorization framework that lets applications access resources on a user's behalf without sharing passwords.
- SAML 2.0
  - XML-based standard for exchanging authentication information between an identity provider and an application.

## SSO와 각 기술의 관계

**SSO(Single Sign-On)**는 한 번 로그인한 뒤 여러 서비스에서 로그인 상태를 이어 쓰는 **목표이자 사용자 경험**입니다. 위 기술들이 모두 SSO의 종류인 것은 아닙니다. 여러 서비스가 같은 인증 체계를 신뢰하도록 연결해야 SSO가 됩니다.

- **Active Directory(AD)**는 회사 계정을 관리하는 서비스입니다. Kerberos 등을 활용해 여러 사내 서비스의 SSO를 구현할 수 있지만, AD를 사용한다는 사실만으로 SSO가 되는 것은 아닙니다.
- **LDAP**은 계정 정보를 조회하는 프로토콜입니다. 로그인에 필요한 계정 확인에 쓰일 수 있지만, LDAP만으로 여러 서비스 사이에 로그인 상태가 공유되지는 않습니다.
- **OAuth 2.0**은 앱에 리소스 접근 권한을 위임하는 프레임워크로, 그 자체는 로그인 또는 SSO 규격이 아닙니다.
- **OpenID Connect(OIDC)**는 OAuth 2.0을 기반으로 로그인한 사용자의 신원을 확인하는 규격입니다. 여러 앱이 같은 인증 제공자(예: Google)를 이용하면 SSO를 구현할 수 있습니다.
- **SAML 2.0**은 인증 정보를 서비스에 전달하는 규격으로, 여러 서비스가 같은 인증 제공자를 신뢰하는 SSO에 사용할 수 있습니다.

## LDAP and Active Directory

**LDAP**은 사용자와 조직 정보가 저장된 디렉터리를 조회하고 수정할 때 사용하는 통신 규칙(프로토콜)입니다. 예를 들어 애플리케이션이 LDAP을 사용해 직원 계정을 찾거나 비밀번호를 확인할 수 있습니다.

**Active Directory(AD)**는 Microsoft의 사용자·컴퓨터 관리 서비스입니다. 회사의 계정과 그룹을 관리하며, LDAP으로 사용자 정보를 조회할 수 있고 Kerberos 등을 이용해 로그인도 처리합니다.

**차이점:** LDAP은 디렉터리와 대화하는 **방법**, AD는 그 방법을 지원하는 **서비스**입니다. LDAP을 지원하는 디렉터리가 모두 AD인 것은 아닙니다. 또한 LDAP으로 계정을 확인하는 것만으로 여러 서비스에 한 번의 로그인으로 접속하는 SSO가 자동으로 구현되지는 않습니다.

## OpenID Connect and OAuth 2.0

**OAuth 2.0**은 사용자가 다른 서비스의 비밀번호를 알려 주지 않고 앱에 제한된 **접근 권한**을 주는 방식입니다. 예를 들어 사진 편집 앱에 내 사진을 읽을 권한만 부여할 수 있습니다. 핵심 질문은 "이 앱이 무엇을 할 수 있나?"입니다.

**OpenID Connect(OIDC)**는 OAuth 2.0을 바탕으로 사용자 **로그인과 신원 확인**을 위한 규칙을 추가한 방식입니다. 로그인 후 발급되는 ID 토큰으로 앱은 누가 로그인했는지 확인할 수 있습니다. 핵심 질문은 "로그인한 사람이 누구인가?"입니다.

**차이점:** OAuth 2.0은 **권한 부여**, OIDC는 **인증**에 초점을 맞춥니다. 따라서 다른 서비스의 계정으로 로그인하는 기능을 구현할 때는 OAuth 2.0만으로 사용자를 식별하려 하지 않고 OIDC를 사용합니다.

### Google 계정으로 내 앱에 로그인하기

Google 로그인을 구현할 때는 **OIDC**를 사용합니다. OIDC는 OAuth 2.0의 인증 코드 흐름을 기반으로 하지만, 로그인한 사람이 누구인지 확인할 때는 **ID 토큰**을 사용합니다.

1. 앱에 Google 로그인을 설정하고, 사용자가 로그인 버튼을 누르면 Google 로그인 화면으로 이동시킵니다. 요청에는 OIDC를 위한 `openid` 범위와 필요한 경우 `email`, `profile` 범위를 포함합니다. 요청과 응답을 안전하게 연결하기 위해 `state`, `nonce`를 저장하고, 공개 클라이언트에서는 인증 코드 탈취를 막는 PKCE를 적용합니다.
2. 사용자가 Google에서 로그인하면 Google이 앱에 인증 코드를 돌려줍니다. 앱은 돌아온 `state`가 보낸 값과 같은지 확인하고, 서버에서 인증 코드를 Google의 토큰 엔드포인트에 전달해 ID 토큰을 받습니다.
3. 앱의 서버는 Google의 공개 키로 ID 토큰의 서명을 확인하고, 발급자(`iss`), 내 앱을 대상으로 발급됐는지(`aud`), 만료 시각(`exp`), 요청할 때 보낸 `nonce`를 검증합니다. 검증된 토큰의 사용자 식별자(`sub`)로 앱의 사용자를 찾거나 등록합니다.
4. 앱은 해당 사용자에게 **앱 자체 세션**을 발급합니다. 이후 앱에 접속할 때마다 Google에 다시 로그인하는 대신 이 세션으로 로그인 상태를 유지합니다.

Google Drive나 Calendar 데이터를 읽는 기능까지 필요하다면 별도로 해당 API의 **접근 권한(scope)**을 요청하고 **액세스 토큰**으로 API를 호출합니다. 액세스 토큰은 데이터 접근용이며, 로그인한 사용자를 확인할 때 ID 토큰 대신 사용하지 않습니다.