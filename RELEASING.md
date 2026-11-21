# NooP 릴리스 배포 체크리스트

이 리포지토리(`/Users/jehee/Dev/NooP-dist`)는 **소스 코드를 공개하지 않고** 배포용 데스크톱 설치 파일(`NooPMac.dmg`, `NooP-Setup.exe`)과 문서만 올리는 배포 전용 저장소입니다. (`.ipa` 파일은 GitHub Release에 업로드하지 않습니다.)

## 1. Mac / iOS / Windows 빌드 명령어

```bash
# 1) Mac 앱 (NooPMac.dmg)
cd /Users/jehee/Dev/NooP/mac-agent
xcodegen generate
xcodebuild -project NooPMac.xcodeproj -scheme NooPMac -configuration Release -derivedDataPath /tmp/NooPMacBuild build

# 2) iOS 앱 (NooP)
cd /Users/jehee/Dev/NooP/ios
xcodegen generate
xcodebuild -project NooP.xcodeproj -scheme NooP -configuration Debug -destination 'generic/platform=iOS' -derivedDataPath /tmp/NooPIOSBuild build

# 3) Windows 앱 (NooP-Setup.exe)
cd /Users/jehee/Dev/NooP/windows-go
GOOS=windows GOARCH=amd64 CGO_ENABLED=0 go build -ldflags "-s -w -H=windowsgui" -o NooPPC.exe .
makensis installer.nsi
```

## 2. GitHub Release 업로드 명령어

```bash
gh release upload v1.1.0 NooPMac.dmg NooP-Setup.exe --repo sungomi/NooP --clobber
```
