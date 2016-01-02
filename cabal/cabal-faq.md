# Cabal FAQ

https://www.haskell.org/cabal/FAQ.html


## Dependencies conflict

    Resolving dependencies...
    cabal: dependencies conflict: ghc-6.10.1 requires process ==1.0.1.1 however
    process-1.0.1.1 was excluded because ghc-6.10.1 requires process ==1.0.1.0

cabal에는 global과 user의 2가지 package db가 있다. 이때 양쪽에 동일한 패키지가
있는 경우 user의 패키지가 global의 패키지를 오버라이드 한다.

예를 들어 globa package db에 아래와 같은 패키지가 있다고 하자.

       A-1
       / |
      /  |
     B-1 |
      \  |
       \ |
        C-1.0.1.0

 이때 user package db에 B-1이 등록을 하고 이 B-1은 C-1.0.1.1로 빌드되었다고 하자.

       A-1
      / \
     /   \
    B-1   \
     \     C-1.0.1.0
       \
        C-1.0.1.1

이제 A-1은 서로 다른 2개 버젼의 C에 의존하게 되었다. cabal-install dependency
resolver는 이것이 발생하기 전에 해결책을 찾도록 설계되었지만 이것은 이미 설치된
패키지들에서 발생되었다.

이는 cabal-install뿐 아니라 ghc --make에도 혼란을 일으킨다. 운이 좋으면 링커
에러가 발생하겠지만 운이 없으면 실행시에 segfault를 만나게 될 것이다.

ghc의 다음 메이저 릴리스에서는 패키지 ABI (Application Binary Interface)를
추적하여 그들의 ABI에 패키지를 슬롯화할 수 있게 하여서 이 문제를 없애고 설치된
패키지들에 보다 안전하고 유연한 관리를 가능하게 할 것이다.

현재의 해결 방법은 'ghc-pkg list'를 실행하여 global과 user package db에 중복된
패키지를 찾아서 user package db의 것을 제거한다. 예를 들어 'process-1.0.1.1'이
양쪽에 등록되어 있어서 제거를 한다면 아래와 같이 입력한다.

    ghc-pkg unregister --user process-1.0.1.1

이때 다른 패키지들을 깰 수 있다는 경우가 나오면 '--force'를 주고 진행한다.
물론, 깨진 패키지들은 다시 빌드가 되어야 한다. 어떤 패키지들이 깨졌는지는 아래와
같이 확인할 수 있다.

    ghc-pkg check

이 문제를 해결하기 위해서는 core package의 업그레이드를 피해라. 최신 cabal-install
에서는 upgrade 명령어가 사용하지 못하도록 되어 있다.


## Hidden packages (a)

    Could not find module `Data.Map': it is a member of package
    containers-0.1.0.0, which is hidden.

이를 해결하기 위해서는 .cabal 파일의 build-depends에 containers를 추가해야 한다.

이 메시지의 원인은 cabal이 ghc에게 빌드를 요청할때 .cabal파일에 명시적으로 나열된
것을 제외하고 모든 가능한 패키지를 무시하도록 하기 때문이다. 이를 'hidden'이라
부른다.

ghc-pkg에서 hide와 expose 패키지 명령어와는 연관이 없다.


## Hidden packages (b)

    Could not find module `Data.Map': it is a member of package
    containers-0.1.0.0, which is hidden.

이는 해당 패키지가 ghc-6.8을 위해서 업데이트되지 않았기 때문이다. ghc-6.8은
base 패키지를 보다 작은 패키지들로 분할하였다. 이 패키지는 새로운 분할된 base
패키지를 사용하도록 업데이트되어야 한다.

단지 빌드에 필요한 패키지를 얻으려면 누락된 패키지를 .cabal 파일의 build-depends
라인에 추가하면 된다. 위 메시지의 경우라면 containers패키지를 build-depends에
추가하면 된다.

패키지 개발자로 새로운 컴파일러에 맞도록 패키지를 업데이트를 원한다면
[패키지 업데이트하기](http://haskell.org/haskellwiki/Upgrading_packages)를 읽어라.


## runProcess: does not exist

    sh: runProcess: does not exist (No such file or directory)

윈도에서 빌드를 하다가 위와 같은 에러가 발생한다면 ./configure 스크립트 실행에
필요한 sh.exe를 찾지 못했기 때문이다.

./configure 스크립트는 윈도우에서 잘 실행되지 못 하기 쉽다. 이때는 MSYS 셸에서
실행을 하면 된다.


## ExitFailure 1

    cabal: Error: some packages failed to install: foo-1.0 failed during the
    configure step. The exception was: exit: ExitFailure 1

이 에러 메시지는 도움이 되지 않는다. 빌드 로그에서 해당 패키지를 빌드하는 부분을
찾아보라.


## runghc Setup complains of missing packages

    $ runghc Setup.hs configure
    Configuring PER-0.0.20...
    Setup.hs: At least the following dependencies are missing:
    time -any && -any

위와 같은 에러 메시지가 표시되나 해당 패키지는 이미 인스톨이 되어 있다.

    $ ghc-pkg list time
    /usr/lib/ghc-6.10.1/./package.conf:
    /home/me/.ghc/x86-linux-6.10.1/package.conf:
        time-1.1.2.4

'runghc Setup.hs configure'의 기본 설정은 --global이다. 그러나 'cabal configure'의
기본 설정은 --user이다. global 패키지는 user package에 의존할 수가 없다. 따라서
cabal로 패키지를 설치하였다면 이것을 다른 패키지에도 사용할 수 있다. 보통은
cabal 대신 'runghc Setup.hs'를 사용할 필요가 없다.

만약 'runghc Setup.hs'를 사용해야 한다면 --user 옵션을 사용해서 user package db를
사용해라. 만약 계속 'runghc Setup.hs'를 사용해야 한다면 cabal 설정파일에서 global
package db를 사용하도록 기본 설정을 바꿀 수도 있다.


## Parsec 2 vs 3

> I just used cabal to install parsec. Why did it give me version 2 rather than 3?

parsec 패키지의 기본 버젼은 2.x이다. parsec 버젼 3는 명시적으로 요청해야 한다.

    $ cabal install 'parsec >= 3'


# Cabal does into an infinite loop / runs out of memory

> I just upgraded to ghc-6.10 and now cabal runs out of memory when I try to install something

    Resolving dependencies...
    cabal: memory allocation failed (requested 2097152 bytes)

이 에러는 cabal-install 0.5.x 버젼을 ghc-6.10과 사용할때 발생한다. 오래된
cabal-install은 base 3이 base4에 의존하는 것을 대응하지 못하여 의존성 무한루프가
발생한다.

만약 ghc-6.8.x를 설치했다면 cabal-install을 업그레이드 하면된다.

    $ cabal install cabal-install --with-compiler=ghc-6.8.3

아니면 ghc-6.10으로 cabal-install을 새로 설치할 필요가 있다. 설치 후에는 $PATH에
따라 올바른 버젼이 실행되는지 확인해라.

    $ cabal --version
    cabal-install version 0.6.2
    using version 1.6.0.2 of the Cabal library


## Internal error: invalid install plan

    $ cabal install leksah
    Resolving dependencies...
    cabal: internal error: could not construct a valid install plan.
    The proposed (invalid) plan contained the following problems:
    The following packages are involved in a dependency cycle leksah-0.10.0.4

cabal-install을 업그레이드 해라

    $ cabal install 'cabal-install >= 0.10'

이 에러는 intra-package dependencies를 허용하는 새로운 cabal의 기능을 사용하는
패키지를 예전 버젼의 cabal-install로 설치할때 발생한다.

예전 버젼은 새로운 기능을 제대로 지원하지 못하므로 문제가 발생한다.




