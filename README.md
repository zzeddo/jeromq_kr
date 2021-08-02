# jeroMQ(ssook)

jeroMQ(ssook: szmq) provides a framework to build a service using C++ based on `ØMQ library (libzmq)`. By referring to the Facebook Message Queue(fbzmq), I have resolved an issue where fbzmq cannot be applied to a multiplatform environment and a performance delay when receiving data.
- It is a lightweight C++ wrapper for the ØMQ library(`libzmq`) that utilizes the latest C++ syntax and strict type checking, and enables various types of message objects to be sent/received as messages without worrying about communication protocols.
- It helps developers to efficiently create application programs with a powerful `synchronous/asynchronous framework` through time limit and signal processing functions.
- Through a series of monitoring tools such as logging and counter, it can be easily added.

jeroMQ(ssook(쑥) : szmq)은 `ØMQ 라이브러리(libzmq)` 기반으로 C++ 개발 언어를 사용하여 서비스를 구축할 수 있는 프레임워크를 제공합니다. 페이스북 메세지큐(Facebook Message Queue : fbzmq) 참조하여, fbzmq가 멀티플레폼 환경에 적용이 불가한 이슈와 데이터 수신시 성능 지연을 해결하였습니다.

-  ØMQ 라이브러리(`libzmq`)에 대한 최신 C++ 구문과 엄격한 유형 검사를 활용하는 경량 C++ 래퍼이며 다양한 형태의 메세지 객체를 통신규약 걱정없이 메세지로써 송/수신 가능하게 합니다.
- 제한시간, 신호처리 기능을 통한 강력한 `동기/비동기 프레임워크`로 개발자가 효율적으로 응용프로그램을 작성할 수 있도록 도워줍니다.
- 일련의 모니터링 도구를 통하여 서비스에 대한 로깅 및 카운터를 쉽게 추가할 수 있도록 합니다.

## Examples

This is an example of converting the basic example (zguide) provided by zeroMQ to Artimisia.

zeroMQ에서 제공하는 기본 예제(zguide)를 Artimisia로 변환한 예제입니다.
* [1장~5장 기본 예제(zguide)](jeroMQ_examples.md)

## Requirements

I built and tested szmq on Windows 10, CentOS7, Ubuntu 20.04, and it has dependencies on the following libraries.
* zeroMQ library (libzmq-4.3.2 or higher), czmq library (4.0.1 or higher)
* C++ library(must be able to support C++14 standard)
  - Visaul Studio 2017 or higher
  - gcc 7.5 version or higher
* Boost library (I used 1.74 for Windows and 1.65 for Linux)

szmq를 원도우 10 및 CentOS7, Ubuntu 20.04에서 빌드 및 테스트 하였으며, 아래의 라이브러리에 대한 의존성이 있습니다.
* zeroMQ 라이브러리(libzmq-4.3.2 이상), czmq 라이브러리(4.0.1 이상)
* C++ 라이브러리(C++14 표준을 지원 가능해야 합니다.)
  - Visaul Studio 2017 이상
  - gcc 7.5 버젼 이상
* Boost 라이브러리(원도우의 경우 1.74을 사용하였으며, 리눅스의 경우 1.65)

## Building 

Visual Studio 2017 creates a project and builds it with a static library. In a Linux environment, Makefile is used.

Visual Studio 2017에서는 프로젝트 생성하여 정적 라이브러리로 빌드를 수행하며, 리눅스 환경에서는 Makefile을 사용합니다.

## License

The MIT license is used in the same way as Facebook Message Queue (fbzmq).

페이스북 메세지큐(Facebook Message Queue : fbzmq)'와 동일하게 MIT 라이센스를 사용합니다.

## Developer 
The developer(PARK JAEDO) applied MOM(Message Oriented Middleware) called hp BASEstar Open in Korea, and ØMQ to replace TIB/Rendezvous (TIB/RV) used in high-tech industry(semiconductor/LCD/OLED). CMW(Control Middleware) technology was transferred from CERN(European Organization for Nuclear Research) and applied in Korea.

If you have any suggestions or inquiries, please email `zzeddo@gmail.com`.

C++ 바인딩(jeroMQ) 제작자(박재도)는 hp BASEstar Open이란 MOM(Message Oriented Middleware)를 국내 적용하였으며 하이텍크 산업(반도체/LCD/OLED)에서 사용하는 TIB/Rendezvous(TIB/RV)를 대체하기 위하여 ØMQ 기반 프로젝트를 수행하였으며 CERN(European Organization for Nuclear Research)의 CMW(Control Middleware) 기술 이전을 받아 국내 적용하기도 하였습니다. 

건의 및 문의 사항 있으실 경우 `zzeddo@gmail.com` 메일 부탁 드립니다.

# The Guide for jeroMQ 

This guide was created by converting to jeroMQ based on the zeroMQ guide.
* [Preface](preface_sook.md)
* [1 - Basics](chapter1_sook.md)
* [2 - Sockets and Patterns](chapter2_sook.md)
* [3 - Advanced Request Reply Patterns](chapter3_sook.md)
* [4 - Reliable Request Reply Patterns](chapter4_sook.md)
* [5 - Advanced Pub Sub Patterns](chapter5_sook.md)
* [Postface](postface_sook.md)

본 가이드는 zeroMQ 가이드를 기반으로 jeroMQ에 변환하여 작성하였습니다.
* [서문(Preface)](preface_sook.md)
* [1장 - 기본(Basics)](chapter1_sook.md)
* [2장 - 소켓 및 패턴(Sockets and Patterns)](chapter2_sook.md)
* [3장 - 고급 요청-응답 패턴(Advanced Request Reply Patterns)](chapter3_sook.md)
* [4장 - 신뢰할 수 있는 요청-응답 패턴(Reliable Request Reply Patterns)](chapter4_sook.md)
* [5장 - 고급 발행-구독 패턴(Advanced Pub Sub Patterns)](chapter5_sook.md)
* [후기(Postface)](postface_sook.md)
