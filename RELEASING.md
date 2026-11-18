# TouchDeck 릴리스 배포 체크리스트 (실수 방지 가이드)

이 리포지토리는 **소스 코드를 공개하지 않고** 배포용 바이너리(`TouchDeckMac.dmg`, `TouchDeck.ipa`, `TouchDeckPC.exe`)와 문서만 올리는 배포 전용 저장소입니다.

## 1. 새 버전 빌드 전 필수 확인사항
1. **iOS `Info.plist` 권한 키 확인** (`ios/project.yml`의 `info.properties`):
   - `NSLocalNetworkUsageDescription`
   - `NSBonjourServices` (`_airpad._tcp`)
   - `NSBluetoothAlwaysUsageDescription`
   - `NSAppTransportSecurity` (`NSAllowsLocalNetworking: true`)
   > 주의: `xcodegen generate` 실행 시 `project.yml`에 `info.properties`가 빠져 있으면 `Info.plist`가 초기화되어 앱이 실행 즉시 크래시(`SIGABRT`)납니다.
2. **개인정보 노출 점검**:
   - 스크린샷을 새로 찍을 때는 반드시 로컬 IP(`192.168.x.x`)와 본명이 포함된 Mac 기기 이름을 마스킹한 뒤 `docs/`에 반영하세요.
   - 이 리포지토리(`/Users/jehee/.gemini/antigravity/scratch/AirPad-dist`)에는 소스 코드(`.swift`, `.py`, `.xcodeproj`)를 절대 커밋하지 마세요.

---

## 2. Mac / iOS Release 바이너리 생성 명령어

```bash
# 1) Mac 앱 Release 빌드 -> AirPadMac.dmg 생성
cd /Users/jehee/.gemini/antigravity/scratch/AirPad/mac-agent
xcodegen generate
xcodebuild -project AirPadMac.xcodeproj -scheme AirPadMac -configuration Release -derivedDataPath build_release CODE_SIGNING_ALLOWED=NO build

rm -rf /tmp/AirPadDMG && mkdir -p /tmp/AirPadDMG
cp -R "build_release/Build/Products/Release/AirPadMac.app" /tmp/AirPadDMG/
ln -s /Applications /tmp/AirPadDMG/Applications
hdiutil create -volname "AirPad Mac" -srcfolder /tmp/AirPadDMG -ov -format UDZO /tmp/AirPadMac.dmg
rm -rf /tmp/AirPadDMG

# 2) iOS 앱 Release 빌드 -> 미서명 AirPad.ipa 생성 (Sideloadly용)
cd /Users/jehee/.gemini/antigravity/scratch/AirPad/ios
xcodegen generate
xcodebuild -project AirPad.xcodeproj -scheme AirPad -configuration Release -sdk iphoneos -derivedDataPath build_release CODE_SIGNING_ALLOWED=NO build

rm -rf /tmp/AirPadPayload /tmp/AirPad.ipa
mkdir -p /tmp/AirPadPayload/Payload
cp -R "build_release/Build/Products/Release-iphoneos/AirPad.app" /tmp/AirPadPayload/Payload/
cd /tmp/AirPadPayload && zip -qry /tmp/AirPad.ipa Payload
rm -rf /tmp/AirPadPayload
```

## 3. Windows `AirPadPC.exe` 빌드 (Windows PC에서 실행)
```bat
cd windows-agent
build_exe.bat
:: 생성 위치: windows-agent\dist\AirPadPC.exe
```

## 4. GitHub Release 업로드 명령어
```bash
# 새 태그로 릴리스 생성 (예: v1.0.1)
gh release create v1.0.1 \
  /tmp/AirPadMac.dmg \
  /tmp/AirPad.ipa \
  --repo sungomi/AirPad \
  --title "AirPad v1.0.1" \
  --notes "변경사항 요약"

# 기존 릴리스(v1.0.0)에 Windows .exe만 추가 업로드할 때:
gh release upload v1.0.0 AirPadPC.exe --repo sungomi/AirPad --clobber
```
