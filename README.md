elixir로 아주 간단한 프로그램을 만들고 터미널에서 `IEx | elixir`로 실행하는 것이 아닌 brew를 통해 패키징된 프로그램으로 만들어 사용할 수 있도록 brew tab을 사용합니다.

## WIP - Elixir로 만든 프로그램 brew로 배포

Elixir로 만든 프로그램을 Homebrew 패키지로 배포하려면, 기본적으로 **바이너리 파일**이나 **escript**로 컴파일한 후 이를 Homebrew Formula로 작성해 Tap 레포에배포하는 방법을 사용합니다.

### 1. Elixir 프로그램을 바이너리 또는 escript로 컴파일

Homebrew를 통해 배포하려면 사용자가 컴파일 없이 쉽게 실행 할 수 있도록 **바이너리 파일**이나 **escript**로 패키징 하는 거이 좋습니다.

#### escript로 패키징

1. `mix.exs` 파일에 `escript` 설정을 추가합니다.

```elixir
defmodule MyApp.MixPorject do
  use Mix.Project

  def project do
    [
      app: :my_app,
      version: "0.1.0",
      elixir: "~> 1.10",
      start_permanent: Mix.env() == :prod,
      deps: deps(),
      escript: [main_module: MyApp.CLI] # main_module을 프로그램의 진입점 모듈로 설정
    ]
  end

  defp deps do
    []
  end
end
```

2. 프로그램을 컴파일하여 escript 파일을 생성합니다.

```shell
mix escript.build
```

이 과정이 완료되면 `./my_app`이라는 escript 파일이 생성됩니다.

### 2. Github Release에 escript 파일 업로드

Homebrew Formula는 URL로 배포 파일을 참조해야하므로, 컴파일된 escript 파일을 Github Release에 업로드 하는 것이 일반적입니다.

1. Github에서 repo에 릴리스를 생성하고, 컴파일한 my_app escript 파일을 **Assets**에 추가합니다.
2. 업로드한 escript 파일의 URL을 복사합니다

### 3. Homebrew Tap repo 생성 및 Formula 작성

현재 저장소에 Formula 파일인 `app.rb`에 해당 내용을 추가합니다.

```rb
class MyApp < Formula
  desc "My Elixir program"
  homepage "https://github.com/<username>/my_app"
  url "https://github.com/<username>/my_app/releases/download/v0.1.0/my_app"
  sha256 "your_calculated_sha256"

  def install
    bin.install "my_app"
  end
end
```

여기서 URl은 Github Release에 업로드한 파일의 URL입니다. sha256은 해당  파일의 SHA-256 해시 값입니다. 해시 값을 확이하려면 터미널에서 다음 명령어를 실행합니다.

```
shasum -a 256 my_app
```

결과로 출력된 해시 값을 Formula에 추가합니다.

### 4. Homebrew 설치 및 테스트

사용자가 Homebrew를 통해 설치할 수 있도록 Tap을 추가하고 Formula를 설치하는 명령어를 안내할 수 있습니다.

1. Tap을 추가합니다.

```sh
brew tap <username>/<tap_name>
```

2. 프로그램을 설치합니다.

```sh
brew install my_app
```

이렇게 하면, Homebrew를 통해 Elixir 프로그램을 쉽게 설치하고 관리할 수 있게 됩니다.
