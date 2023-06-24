---
title: 3. Option 몇번째
date: '2021-01-01'
type: book
weight: 40
tags:
  - current
---

# 예제 코드
다음은 단위 변환기의 예제 코드입니다:

```python
import click

@click.command()
@click.option('--length', type=float, help='Length value to convert')
@click.option('--from-unit', type=click.Choice(['m', 'cm', 'mm']), help='Unit to convert from')
@click.option('--to-unit', type=click.Choice(['m', 'cm', 'mm']), help='Unit to convert to')
def length_converter(length, from_unit, to_unit):
    """Convert length between different units"""
    conversion_factors = {
        'm': 1,
        'cm': 100,
        'mm': 1000
    }
    result = length * conversion_factors[from_unit] / conversion_factors[to_unit]
    click.echo(f'{length} {from_unit} is equal to {result} {to_unit}')

if __name__ == '__main__':
    length_converter()

```

## 함수 설명
이 함수는 @click.command() 데코레이터로 데코레이트되어 커맨드라인 인터페이스의 진입점이 됩니다. 함수의 인자로는 단위 변환에 필요한 값들을 받습니다.

@click.option() 데코레이터를 사용하여 length, from-unit, to-unit 세 가지 옵션을 정의합니다. 각각은 -- 접두사를 사용하여 커맨드 라인에서 입력할 수 있는 옵션으로 만들어집니다. type 매개변수를 사용하여 입력된 값의 타입을 지정할 수 있습니다. click.Choice()를 사용하여 from-unit과 to-unit 옵션의 값으로 'm', 'cm', 'mm' 중 하나만 선택할 수 있도록 제한합니다. help 매개변수는 각 옵션에 대한 도움말 메시지를 정의합니다.

length_converter 함수에는 docstring으로 작성된 함수 설명이 있습니다. 이 설명은 Click에 의해 자동으로 커맨드 라인 도움말로 사용됩니다.

conversion_factors 딕셔너리는 단위 간 변환에 사용되는 계수를 저장합니다. length, from_unit, to_unit을 사용하여 변환을 수행하고, 결과를 result 변수에 저장합니다.

마지막으로, click.echo() 함수를 사용하여 변환 결과를 출력합니다. 출력 메시지에는 변환되는 값과 단위가 포함됩니다.

마지막 부분은 스크립트가 직접 실행될 때만 length_converter() 함수를 호출합니다. 이렇게 함으로써 모듈을 다른 스크립트에서 임포트했을 때 함수가 자동으로 실행되지 않습니다.


## 도움말 메시지 출력

이 프로그램을 실행하면서 사용자가 도움말 메시지를 볼 수 있는 방법을 설명해보겠습니다. 도움말 메시지는 프로그램의 기능, 옵션 및 사용법에 대한 자세한 설명을 제공합니다.

터미널에서 다음과 같이 실행하면 도움말 메시지를 볼 수 있습니다.

```bash_session
python length_converter.py --help`
```

위 명령을 실행하면 옵션에 대한 설명과 사용법이 출력됩니다.

```bash_session
Usage: length_converter.py [OPTIONS]

  Convert length between different units

Options:
  --length FLOAT    Length value to convert
  --from-unit [m|cm|mm]
                    Unit to convert from
  --to-unit [m|cm|mm]
                    Unit to convert to
  --help            Show this message and exit.

```

help, type 매개 변수와 함수의 docstring들로 충분히 좋은 help 메세지를 보여줍니다.


## 올바른 사용 예시

이제 사용자가 올바른 값을 입력했을 때 단위 변환을 수행하는 방법을 설명해보겠습니다. 사용자는 --length, --from-unit, --to-unit 옵션을 지정하여 원하는 단위 변환을 실행할 수 있습니다.

터미널에서 다음과 같이 프로그램을 실행해보세요:

```
$ python length_converter.py --length 10 --from-unit cm --to-unit mm
10.0 cm is equal to 100.0 mm
```

## 잘못된 값 입력

```bash_session
python length_converter.py.py --length 10 --from-unit inch --to-unit m
위 명령을 실행하면 다음과 같은 오류 메시지가 출력됩니다:
```
```bash_session
Error: Invalid value for '--from-unit': 'inch' is not one of 'm', 'cm', 'mm'.
```

이 오류 메시지는 사용자가 --from-unit 옵션에 잘못된 값을 입력했음을 알려줍니다.

이와 같이 사용자가 잘못된 옵션 또는 값 입력 시 Click가 자동으로 오류 메시지를 출력해주는 기능을 이용하여 사용자가 올바른 값을 입력할 수 있도록 안내할 수 있습니다.
