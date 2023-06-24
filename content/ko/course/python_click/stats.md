---
title: 3. Option 몇번째
date: '2021-01-01'
type: book
weight: 40
tags:
  - current
---

```
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

type 매개변수는 사용자의 입력한 값이 어떤 형태인지 확인하는 옵션입니다.
기본적으로 float,int(integer),str(string),bool을 지원합니다.
특정 값들의 리스트에서만 입력을 받고 싶다면 `click.Choice`를 이용하면 됩니다.
`--from-unit`의 type처럼 
 `type=click.Choice(['m', 'cm', 'mm'])`로 사용하면 m, cm, mm의 형태로만 받아야 합니다.
사용자가 type과 맞지 않는 값을 입력하면 에러가 발생합니다.
help 매개변수는 각 옵션의 help 메세지를 정의하는 것입니다.

함수 아래에서는 option명대로 변수가 들어가게 됩니다.
length옵션으로 받은 입력은 length 변수에 들어가게 됩니다.

