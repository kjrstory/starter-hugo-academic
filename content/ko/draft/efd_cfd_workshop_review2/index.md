---
title: EFD-CFD 워크샵 논문 리뷰2
date: 2023-05-25T07:22:43.471Z
draft: false
featured: false
authors:
  - admin
categories:
  - CFD
tags:
  - CFD
  - OpenFoam
---
  
# 5. OpenFoam
그리고 오픈소스에서 하나쯤은 넣어야겠다고 생각했습니다. 
CFD분야의 오픈소스는 OpenFoam이 가장 유명합니다.
하지만 OpenFoam을 논문에 포함하진 못했는데요.
이 당시에는 OpenFoam이 익숙하지 않았었고 제가 워크샵에서 선택했던 케이스가 해석이 잘 안되었습니다.
구체적으로 이야기하면 마하수가 높아 압축성 솔버가 필요한데 OpenFoam의 압축성 솔버는
수렴이 잘 되지 않았던 것으로 기억을 합니다.


논문에 들어간 코드는 Fluent, CFX, Star-CCM+, SU2입니다.
하지만 제가 검토했던 코드는 더 있었습니다.

# 6. Acusolve
먼저 구조해석분야에서 유명한 Altair Hyperworks 계열의 Acusolve입니다.
Acusolve와 다른 코드와의 가장 큰 차이점은 FEM이란 것입니다. 
FEM은 보통 구조해석에 많이 쓰이고 FVM은 유체해석에 많이 쓰이는데 
Hyperworks는 구조해석 SW가 더 먼저 하다보니 CFD코드도 FEM으로 한게 아닌가 싶습니다.
Acusolve도 본래 Hyperworks를 위해 다른 연구원이 구매한 SW입니다만 
통합 라이센스제도라 Acusolve도 사용이 가능하였고 호기심이 많았던 제가 사용했었습니다.
단 결국 못들어갔던건 마하수 제한때문입니다.
Acusolve 솔버는 그 때 당시 약 마하수 0.3정도에서만 해석이 가능했던 솔버였습니다.
현재는 이 제한이 일부 해소 된것으로 알고 있습니다.


# 7. Comsol
다음은 Comsol입니다.
Comsol은 멀티피직스를 특성화시킨 SW입니다.
다른 SW들에 비해 사용량이 많지는 않으나 몇몇 분야에서는 많이 사용하는것 같습니다.
이 역시 라이센스를 하나 보유하고 있었고 역시 튜토리얼까지는 했었습니다.
그러나 사용법이 익숙하지 않아 포기했었던 SW입니다.

정리하면 저는 7개의 코드를 해석하려고 했습니다. 
CFD분야가 아니라면 감이 안 오시겠지만 CFD를 해보셨다면 7개의 해석 SW를 한명의 연구원이 한다는것이
말도 안된다고 바로 튀어나오는 말입니다.

보통의 CFD하시는 분들은 대학원때는 랩의 인하우스 코드를 쓰다가 회사에 와서는 상용 코드를 많이 쓰게 됩니다.
대부분 상용 SW는 하나를 주력으로 정하고 가끔 열정적이신분은 부로 돌릴만한 코드를 더 공부하긴 합니다.
저처럼 4개를 논문에 쓰고 처음 목적은 7개였다는건 무모한 도전에 가깝습니다.
왜냐하면 CFD사람들이 게을러서 그런건 아니고 1-2개 고급 실력까지 익히는것도 매우 힘들기 때문입니다.
이렇게 많은 코드를 하려고 했다는 것은 오히려 하나도 제대로 못다루면서 이것저것 해보는 사기꾼 취급을 받을 수 있기 때문에 조심해야 합니다. 
