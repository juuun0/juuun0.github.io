---
title: "Airbone: Zero-Click RCE in Apple AirPlay"
category: Blog
tags: ['Trend', 'Real World']
date: 2025-05-31 21:30 +0900
author: ji9umi
---

## 0x01. Agenda

Oligo Security Research는 Apple사의 AirPlay 프로토콜과 SDK에서 여러 취약점을 발견하였으며 이는 **Airbone** 이라는 이름으로 명명되었다. 이 취약점을 통해 달성할 수 있는 목표는 다음과 같다:

1. Zero-Click RCE
2. One-Click RCE
3. ACL and user intercation bypass
4. Local Arbitrary File Read
5. Sensitive information disclosure
6. MITM attacks
7. DoS

해당 취약점은 AirPlay SDK를 사용하는 Apple 기기와 third-party 제품도 영향을 받기 때문에 광범위한 피해를 입힐 수 있다. Oligo는 CVE-2025-24252와 CVE-2025-24132를 활용하면 공격자가 악성 zero-click RCE를 무기화하여 사용할 수 있다고 밝혔다.

## 0x02. Technical Analysis

> 현재 공개된 문서는 언급된 취약점 내용 중 RCE에 대해 다루고 있으며 이외의 내용은 추후 공유할 수 있다고 함.

- macOS
> - CVE-2025-24252 & CVE-2025-24206 Chaining ⇒ Zero-Click RCE
> - CVE-2025-24271 & CVE-2025-24137 Chaining ⇒ One-Click RCE
- AirPlay SDK
> - CVE-2025-24132 ⇒ Zero-Click RCE
- Car-Play Devices
> - CVE-2025-24132 ⇒ Zero-Click & One-Click RCE
> > - 연결 조건, 제조사에 따라 구현 방식에 차이가 있어 동일한 취약점을 사용해도 다른 방식의 접근이 요구되는 것으로 보인다.

### macOS zero-click RCE

**CVE-2025-24252**는 use-after-free 취약점으로 UAF로 사용되거나 공격자가 macOS 기기에 원격 코드 실행을 하기 위한 “arbitrary free” 동작으로 사용될 수 있다. 이는 user interaction bypass 취약점인 **CVE-2025-24206**과 연계하여 zero-click RCE를 달성할 수 있다.

공격 가능한 대상에는 일부 제약이 존재하는데 ***공격자와 동일한 네트워크***에 있어야 하며 AirPlay receiver가 활성화 되어 있는 사용자 중 “**Anyone on the same network”** 또는 “**Everyone”** 으로 설정되어 있어야 한다. 조건이 만족되는 경우 공격자는 피해자의 별도 동작을 요구하지 않아도 다른 기기로 확산하는 것이 가능하다.

원 글에서는 이 조건을 만족하는 시나리오를 다음과 같이 작성하였다:

> 피해자의 기기는 사전에 공공 WiFi를 사용할 때 장악된다. 이후 피해자가 본인의 직원망 등 다른 네트워크에 접속할 경우 이는 공격자에게 내부의 다른 기기에 접근 가능한 경로를 제공한다.

이 취약점을 재현하기 위해 “write-what-where” primitive를 사용하여 macOS의 기본 음악 앱을 덮어씌운다. 익스플로잇 코드 실행 이후 사용자가 음악 앱을 누르면 임의 이미지를 실행하는 것을 통해 증명하고 있다.

덧붙여 익스플로잇에 사용된 primitive가 대상의 어떤 메모리 주소든 접근이 가능한 것이었기 때문에 이와 같이 진행된 것이며 다른 primitive를 활용하면 앱을 여는 것을 요구하지 않고도 가능할 것이라고 전하였다.

### macOS one-click RCE

**CVE-2025-24271**은 ACL에서 발생하는 취약점으로 공격자가 페어링하지 않은 상태에서 AirPlay 명령어를 보내는 것이 가능하게 한다. 이 취약점이 **CVE-2025-24137**과 연계되면 공격자와 동일한 네트워크에 있는 macOS 기기 중 AirPlay recever 설정이 **”Current User”**로 되어있을 때 one-click RCE 공격이 가능하다.

> CVE-2025-24137은 2025년 1월 27일 macOS Sequoia 15.3에서 패치된 취약점으로 type confusion으로 인해 예기치 않은 앱 종료 혹은 임의 코드 실행이 가능한 취약점이다.

### Car-Play Devices zero-click and one-click

**CVE-2025-24132**는 스택 기반 버퍼 오버플로우 취약점으로 특정 조건에 부합하는 경우 zero-click RCE로 사용될 수 있다. CarPlay를 대상으로 공격하여 얻을 수 있는 기대 효과는 이미지를 띄우거나 오디오를 재생하여 운전자의 주의를 산만하게 만들거나, 대화 내용 감청 또는 차량의 위치 추적 등이 가능하다.

1. **WiFi hotspot**을 사용하는 환경에서는 인접한 CarPlay 기기가 예측 가능하거나 알려진 패스워드로 연결된 경우 접근 권한을 얻고 RCE로 이어질 수 있다.
2. 일부 CarPlay 기기는 WiFi credentials을 교환하기 위해 **Bluetooth over the IAP2** 프로토콜을 사용하고 페어링 과정에서 PIN 입력을 요구한다. 공격자는 대상 기기와 근접한 위치에 있으며 PIN 입력 및 확인이 가능한 경우 RCE 공격이 가능하다. 일부 케이스에서는 피해자의 click이 요구될 수 있기 때문에 one-click RCE로 동작한다.
3. 무선 환경을 지원하지 않고 **USB**와 같이 물리적 연결이 필요한 기기는 이를 통해 취약하다.

## 0x03. Technical Overview

AirPlay는 HTTP와 RSTP 프로토콜을 혼합한 형태로 7000번 포트를 사용하여 통신한다. 이 시스템에서는 각종 명령어나 추가적인 파라미터를 *plist* 형식으로 인코딩하여 주고받는데 이는 **property list**의 약자로 Apple 생태계에서 사용되는 `key - value` 구조의 형식이다.

Apple Core Foundation에서 중요한 역할을 하는 만큼 이를 잘 이해하는 것을 바탕으로 관련된 취약점을 발굴하는데 도움이 될 수 있다. 본 글에서는 *plist* 파라미터를 부적절하게 제어하여 발생하는 **CVE-2025-24129** type confusion 취약점을 예시로 설명하였다.

> 세부적인 내용은 https://www.oligo.security/blog/airborne 참고

## 0x04. Conclusion

위에 기술한 내용 외에도 패치되었으나 CVE가 발급되지 않은 2개의 취약점에 대한 내용을 추가로 다룬다. 또한 AirPlay를 분석하게 된 계기로 별도의 글이 작성되어 있는데, 제목에서 유추할 수 있듯이 로컬 네트워크 상에서 0.0.0.0으로 열어둔 서비스를 대상으로 외부에서 트리거하는 방식에 대한 연구 내용이 담겨있다.

원격에서 접근이 불가능하다고 판단하여 개발을 진행할 경우 다른 취약점과 연계될 경우 그 파급력이 커질 수 있기 때문에 훑어보면 좋을 것 같다.

---

## 0xFF. References

- Agenda
> - [Airbone: Wormable Zero-Click RCE in Apple AirPlay](https://www.oligo.security/blog/airborne)
- Technical Analysis
> - macOS zero-click RCE
> > - [Write-what-where(Arbitrary Memory Overwrite)](https://www.lazenca.net/pages/viewpage.action?pageId=25624658)
> > - [Write-what-where condition](https://cwe.mitre.org/data/definitions/123.html)
- Conclusion
> - [0.0.0.0 Day: Exploiting Localhost APIs From the Browser](https://www.oligo.security/blog/0-0-0-0-day-exploiting-localhost-apis-from-the-browser)
