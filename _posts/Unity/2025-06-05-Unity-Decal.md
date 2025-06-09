---
title: "[Unity] 데칼과 적용할 레이어 선택
author: RePro
date: 2025-06-05 10:00:00 +0900
categories: [Programming, Unity]
tags: [unity]
---

## 데칼 적용할 레이어 선택

![Image][step1]

Project Settings -> Graphics 에서 현재 적용된 랜더 파이프라인 에셋을 확인할 수 있다.

랜더 파잉프라인 에셋에서는 다시 랜더러 에셋을 찾을 수 있다.

---

![Image][step2]

이중에 데칼의 `Use Rendering Layers` 옵션을 활성화 해야 랜더링할 레이어를 선택할 수 있다. 그러지 않으면 데칼 프로젝터에서 설정을 바꿔도 적용되지 않는다.

---

![Image][step3]

Project Settings -> Rendering Layers 에서 데칼용 레이어를 추가한다.


![Image][step4]

이제 UPR Decal Projector에서 Rendering Layers를 선택한다.

![Image][step5]

Mesh Renderer -> Additional Settings -> Rendering Layer Mask <br>
그리고 데칼을 적용하고 싶은 메쉬에서 데칼 레이어를 추가한다. 




[step1]: https://github.com/user-attachments/assets/b251ceaa-af0b-45f6-95ff-a70d8bc54584
[step2]: https://github.com/user-attachments/assets/5544c467-90b2-47e9-b0b7-fc29f5cb9e61
[step3]: https://github.com/user-attachments/assets/141b44e8-75e3-4681-967f-770ce2271534
[step4]: https://github.com/user-attachments/assets/332818b0-47c2-4579-8a8a-cabb8557ef92
[step5]: https://github.com/user-attachments/assets/e7292553-0152-41cf-9e37-034249809062