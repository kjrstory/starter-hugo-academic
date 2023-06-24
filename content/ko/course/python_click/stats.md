---
title: 3. Option 몇번째
date: '2021-01-01'
type: book
weight: 40
tags:
  - current
---

이 강의에서는 Python Click 라이브러리를 사용하여 길이 변환기를 개발하는 방법을 알아보겠습니다. 길이 변환기는 사용자로부터 길이 값과 단위를 입력받아 다른 단위로 변환하는 기능을 제공합니다.

# 예제 코드
다음은 길이 변환기의 예제 코드입니다:

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

## 옵션 설명
type 매개변수는 사용자의 입력한 값이 어떤 형태인지 확인하는 옵션입니다.
기본적으로 float,int(integer),str(string),bool을 지원합니다.
특정 값들의 리스트에서만 입력을 받고 싶다면 `click.Choice`를 이용하면 됩니다.
`--from-unit`의 type처럼 
 `type=click.Choice(['m', 'cm', 'mm'])`로 사용하면 m, cm, mm의 형태로만 받아야 합니다.
사용자가 type과 맞지 않는 값을 입력하면 에러가 발생합니다.
help 매개변수는 각 옵션의 help 메세지를 정의하는 것입니다.

함수 아래에서는 option명대로 변수가 들어가게 됩니다.
length옵션으로 받은 입력은 length 변수에 들어가게 됩니다.


## 도움말 메시지 출력
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


## 에러메세지 출력

length 옵션은 float타입만 받기로 하였는데 float로 변환이 불가능한 문자가 들어가면 아래와 같이 에러메세지를 출력합니다.

```bash_session
$ python length_converter.py --length abc --from-unit cm --to-unit mm
Error: Invalid value for '--length': 'abc' is not a valid float.
```

click.Choice타입을 사용한 옵션에서 리스트에 벗어나는 입력을 넣게 되면 역시 에러메세지를 출력합니다.
```bash_session
$ python length_converter.py --length 10 --from-unit inch --to-unit mm
Error: Invalid value for '--from-unit': invalid choice: inch. (choose from m, cm, mm)
```

## 올바른 사용 예시

제대로 된 사용은 아래와 같습니다.
```
$ python length_converter.py --length 10 --from-unit cm --to-unit mm
10.0 cm is equal to 100.0 mm
```

