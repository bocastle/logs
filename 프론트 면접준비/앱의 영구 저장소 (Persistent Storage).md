# 📱 앱에서는 어떤 저장소를 사용할까?

웹에서는 `localStorage`와 `sessionStorage` 같은 브라우저 내장 저장소를 사용하지만, 앱에서는 플랫폼에 따라 다양한 방식의 저장소가 제공됩니다. 아래에서 각각의 대응 개념과 사용 사례를 살펴봅니다.

---

## 1. 📦 앱의 영구 저장소 (Persistent Storage)

**→ 웹의 `localStorage`에 대응**

- **iOS**: `UserDefaults`, `Core Data`, `FileManager`, `Keychain` 등
- **Android**: `SharedPreferences`, `Room`, `File storage`, `EncryptedSharedPreferences` 등
- **크로스플랫폼(Flutter, React Native 등)**: `AsyncStorage`, `SecureStorage`, `Hive`, `MMKV` 등 사용

✅ 앱이 종료되어도 데이터가 유지됩니다.  
✅ 설정값, 로그인 토큰, 앱 상태 저장 등에 사용됩니다.

---

## 2. 🧠 세션/임시 메모리 저장소 (In-Memory / Temp Storage)

**→ 웹의 `sessionStorage`나 상태 관리 도구에 대응**

- 앱 실행 중에만 유지되며, 앱을 종료하면 사라지는 형태입니다.
- 예: 전역 변수, 전역 상태 관리 (`Redux`, `Zustand`, `Provider`, `Context`, `MobX` 등)

✅ 빠른 접근이 가능하지만 휘발성입니다.  
✅ 화면 간 데이터 전달이나 인증 직후 상태 보관 등에 활용됩니다.

---

## 3. 🔐 보안을 고려한 저장소

민감한 정보(예: 액세스 토큰, 비밀번호 등)는 단순한 저장소에 저장하지 않고, 보안 전용 저장소를 사용해야 합니다.

- **iOS**: `Keychain`
- **Android**: `EncryptedSharedPreferences`, `Keystore`
- **크로스플랫폼**: `react-native-keychain`, `flutter_secure_storage`

👉 웹에서 말하는 "HTTP-Only 쿠키"처럼, 앱에서도 **보안 전용 저장소를 사용해야 합니다**.

---

## 🧾 요약

| 저장소 종류 | 웹                    | 앱                                                     |
| ----------- | --------------------- | ------------------------------------------------------ |
| 영구 저장소 | `localStorage`        | `SharedPreferences`, `UserDefaults`, `AsyncStorage` 등 |
| 임시 저장소 | `sessionStorage`      | 메모리 변수, 상태 관리 라이브러리                      |
| 보안 저장소 | HTTP-Only 쿠키 (서버) | `Keychain`, `EncryptedStorage` 등                      |
