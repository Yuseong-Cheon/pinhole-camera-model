# 핀홀 카메라 모델

3D 비전 기초를 한 단계씩 풀어 보는 인터랙티브 공부 페이지다. 각 쪽은 "앞 단계의 문제 → 이 단계의 방법 → 남은 문제" 순서로 이어진다.

## 쪽 구성

| 쪽 | 내용 | 끝의 문제 → 다음 쪽 |
|---|---|---|
| ⓪ 로드맵 | 사진에서 면(mesh)까지: 재료 얻기(①–⑥)와 Mesh 만들기(⑦, 부록 A·B), Fast-FoundationStereo·GGGS가 채우는 칸 | 기초부터 ①로 |
| ① 외부 파라미터 [R \| t] | 월드 좌표계 → 카메라 좌표계 변환, t = −R C, C = −Rᵀ t | 이미지는 픽셀 단위다 |
| ② 내부 파라미터 K | 카메라 좌표 → 이미지 좌표 (원근 투영), u = f · x_c / z_c + c_x | 같은 광선 위의 점은 같은 픽셀 (깊이 모호성) |
| ③ 삼각측량 · 스테레오 | 두 광선의 교점, Z = f · b / d | 대응점을 찾아야 한다 |
| ④ 특징점 검출 · 매칭 | 확대 이미지 (평면 · 엣지 · 코너), SIFT 비율 검사 | [R \| t]를 모르면 에피폴라 선을 못 그린다 |
| ⑤ 8점 알고리즘 · RANSAC | 에피폴라 선 개념도, 대응점 8쌍 이상으로 기본 행렬 F 추정(x_2ᵀ F x_1 = 0), 무작위 8쌍 + 투표로 틀린 짝 거르기, F → E → R, t, 끝에 단계 정리 | 사진 쌍마다 길이 단위가 달라 여러 장을 그대로 이어 붙일 수 없다 |
| ⑥ SfM · MVS | SfM 단계 개념도: 기준 세우기 → PnP → 새 점 삼각측량 → 반복, 에피폴라 선 위 패치 비교 → 깊이 지도 | 깊이 지도는 사진마다 따로 있고 구멍과 잡음이 있다 |
| ⑦ TSDF · Marching Cubes | 깊이 지도 11장을 복셀에 평균(TSDF), 부호가 바뀌는 변에서 표면 추출(Marching Cubes) | 민무늬·반짝이는 면의 구멍과 울퉁불퉁 → 부록 A 렌더링으로 풀기 |
| 부록 A 렌더링으로 풀기 | NeRF와 Gaussian Splatting의 색 누적·픽셀 비교·학습, NeuS와 최근 면 추출 연구 | — |
| 부록 B 최근 연구 (2026) | GGGS의 연속 깊이, Gaussian Wrapping의 빈 정도와 닫힌 선, MeshSplatting의 창 함수와 제한 Delaunay | — |

①–③은 바닥만 본 2D 개념도다. 실제 이미지는 DTU scan 114의 시점 13 / 32(④–⑤, ⑥ MVS)와 39–49(⑦)를 쓴다.

## 여는 법

`camera_model_study.html`과 `data/` 폴더를 같은 위치에 두고 연다. 쪽은 아래 버튼, 위의 ⓪–⑦·부록 단추, 키보드 ←→로 넘긴다.

수식은 HTML 안에 LaTeX(`\( ... \)`, `\[ ... \]`)로 직접 쓴다.

④의 확대 이미지도 파일을 직접 열어(`file://`) 볼 수 있다. 로컬 서버로 열어도 된다.

```bash
python3 -m http.server 8000
# http://localhost:8000/camera_model_study.html
```

## 데이터 출처

- **Fast-FoundationStereo (⓪):** Bowen Wen, Shaurya Dewan, Stan Birchfield (NVIDIA), CVPR 2026, [arXiv 2512.11130](https://arxiv.org/abs/2512.11130) (공개 2025-12-11). 정렬된 왼·오 사진 쌍에서 시차 지도를 구한다. FoundationStereo를 교사로 지식 증류했으며, FoundationStereo보다 10배 넘게 빠르다.
- **FoundationStereo (⓪):** Bowen Wen 외, CVPR 2025, [arXiv 2501.09898](https://arxiv.org/abs/2501.09898). 합성 학습 데이터 100만 쌍을 사용했다.
- **COLMAP (⓪):** Schönberger & Frahm, *Structure-from-Motion Revisited*, CVPR 2016; Schönberger 외, *Pixelwise View Selection for Unstructured Multi-View Stereo*, ECCV 2016.
- `data/fast_foundationstereo_fig1.png`: Wen 외, Fast-FoundationStereo (CVPR 2026) 그림 1, [arXiv 2512.11130](https://arxiv.org/abs/2512.11130), CC BY 4.0. Claude가 300 dpi로 잘라 1400 px로 줄임.
- **연구실 실험 영상 (⓪ 로드맵):** `data/gggs_vs_3dgs.mp4`는 주점 보정 후 3DGS와 GGGS의 색·형상 비교, `data/gggs_depth_loss.mp4`는 GGGS와 측정 깊이 손실을 더한 GGGS 비교다. 심영찬의 실험 영상으로, 연구실 그룹 미팅 Canva 발표 20·21쪽에서 영상 부분만 잘라냈다.
- **Dense-SfM 그림(①):** JongMin Lee, Sungjoo Yoo, "Dense-SfM: Structure from Motion with Dense Consistent Matching", CVPR 2025. 그림 1 아래 칸 "Textureless – Object"를 잘라 실었다(`data/dense_sfm_fig1_textureless_object.png`). [arXiv 2501.14277](https://arxiv.org/abs/2501.14277), CC0.
- **DTU MVS 2014:** Aanæs et al., IJCV 2016; Jensen et al., CVPR 2014. https://roboimagedata.compute.dtu.dk/?page_id=36
  - scan 114, 조명 3, rectified 1600×1200 이미지 2장(시점 13, 32)을 JPEG로 저장해 실었다.
  - 기준 F와 R, t 비교값은 DTU 보정 카메라 행렬로 계산했다(`data/dtu_cam.js`: K_1, K_2, 시점 13 → 32 상대 자세).
- **대응점:** OpenCV SIFT(nfeatures = 6500) + FLANN, 비율 검사 0.75. 특징점은 이미지 1에서 4939개, 이미지 2에서 4772개이고, 비율 검사를 통과한 대응점은 624쌍이다. RANSAC 투표 문턱은 이미지 2의 에피폴라 선까지 거리 2 픽셀이다. 정상과 틀린 대응점은 기준 F에서 대칭 에피폴라 거리 2 픽셀을 문턱으로 나눴다.

- `data/sift_8bins.png`: Claude가 그린 SIFT Descriptor 8방향 상자 개념도(원본 `../scripts/sift_hist_figure.html`). 밝기는 손으로 정한 합성 예시이며 실제 사진이 아니다. Lowe, IJCV 2004를 참고했다.
- `data/rt_four_solutions.png`: Claude가 그린 (R, t) 후보 4쌍 개념도(원본 `../scripts/rt_four_solutions_figure.html`). 위에서 본 2D 예시 배치이며 실제 데이터가 아니다. Hartley & Zisserman, *Multiple View Geometry* 9.6절 그림 9.12를 본떴다.

- **SfM (⑥):** Claude가 그린 SfM 단계 개념도(`data/sfm_steps.png`, 원본 `../scripts/sfm_steps_figure.html`).
- **MVS (⑥):** 시점 13 기준.
  - 한 픽셀의 닮음 곡선: 시점 32와 15×15 패치 NCC, 깊이 480–800 mm를 0.5 mm 간격으로 비교.
  - 깊이 지도 `data/mvs_depth.png`: 옆 사진 4장(시점 14, 12, 11, 28), 400×300으로 줄여 깊이 160단계 × 7×7 NCC, 4장 중 가장 닮은 2장 평균. NCC > 0.6, 밝기 분산 > 4, 40픽셀 이상 조각만 남겼다.
  - 비교 기준: DTU 레이저 스캔 점 구름(`stl114_total.ply`). 계산에는 쓰지 않고 비교에만 썼다. 깊이 오차 중앙값 0.91 mm, 깊이를 정한 픽셀 71.1%.

- **TSDF · Marching Cubes (⑦):** DTU scan 114 시점 39–49의 사진 11장과 DTU 보정 카메라(cal18)를 쓴다. 깊이 지도는 각 사진을 기준으로 가까운 이웃 4장과 ⑥처럼 평면 쓸기 NCC로 계산했다(400×300, 깊이 160단계, 7×7 패치, 상위 2장 평균, NCC > 0.6, 분산 > 4, 40픽셀 이상 조각).
  - `data/tsdf_steps.png`: Claude가 그린 TSDF 개념도(원본 `../scripts/tsdf_steps_figure.html`). 숫자와 단면은 예시이며 DTU 데이터가 아니다.
  - `data/mc_why_sign.png`: Claude가 그린 Marching Cubes 안/밖 부호 개념도(원본 `../scripts/mc_why_sign_figure.html`). 값은 손으로 정한 예시이며 실제 데이터가 아니다.
  - 2 mm 복셀, 자르기 거리 10 mm, 관측마다 가중치 1로 평균한다. 레이저 스캔은 받침대를 제외한 물체 상자와 정답 비교에 쓰며, 깊이 지도는 사진에서 구한다.
  - `data/tsdf_mc.js`: ⑥의 카메라 호 평면과 같은 방향에서 본 중앙 높이 단면, 128×128 격자, 1–11장 누적값. 거리/τ를 int8로 양자화(−127…127), −128은 관측 없음이다. 카메라 방향과 레이저 단면 점 300개도 담았다.
  - **칸 하나:** 3D 정육면체를 끌어서 돌리고 꼭짓점을 눌러 부호를 바꾼다. 256가지 배치를 정육면체 회전 24가지와 부호 반전으로 묶어 기본형 1–15를 계산하며, 버튼으로 각 대표 배치를 고른다. 부호가 대각선으로 엇갈리는 애매한 면은 노란색으로 표시한다. 방향키로 꼭짓점을 고르고 Enter·공백으로 부호를 바꿀 수도 있다.
  - **칸 27개:** 4×4×4 격자점(각 축 −3, −1, 1, 3)에서 구의 부호 거리 `f(p) = |p − c| − R`, `c = (−0.5, −0.5, −0.5)`, `R = 3`을 계산한 **구 표면 예시 (실제 데이터 아님)**이며 DTU 데이터가 아니다. 칸마다 다른 색으로 삼각형을 그리고, 같은 변의 점을 선형 보간 `t = v0 / (v0 − v1)`으로 구한다. 끌거나 방향키로 돌리고, ‘칸 떼어 보기’로 이웃 칸이 공유하던 검정 점이 나뉘는 모습을 본다.
  - **한 번에 보기:** 예시 도넛(실제 데이터 아님)을 위·아래 고도 ±25°의 카메라 8대(각 4대)의 모의 깊이 지도로 관측하고, 거리값 누적과 Marching Cubes를 한 장면에서 단계별로 본다. 깊이 지도의 촬영 범위는 격자 전체를 덮으며, 물체를 빗나간 광선은 먼 배경까지 빈 공간으로 관측한다. 합친 장수·보는 층을 바꾸고 끌거나 방향키로 돌릴 수 있다.
  - 3D 데모의 삼각형 표: Paul Bourke, ["Polygonising a scalar field" (1994)](https://paulbourke.net/geometry/polygonise/)의 `triTable`. 로컬 PyTorch3D `ops/marching_cubes_data.py`의 `FACE_TABLE` 사본에서 가져왔으며, 꼭짓점·변과 경우 번호는 Bourke 번호를 쓴다. 외부 라이브러리 없이 기존 Canvas 2D에서 정사영·회전·깊이 정렬·양면 음영을 계산한다.
  - 재생성: 작업 폴더에서 `nice -n 10 /home/cheon/Desktop/MJMJ/experiments/contact_mesh/.venv-gs2mesh/bin/python scripts/make_tsdf_mc.py`. 숫자는 `../scripts/tsdf_mc_report.json`, 단면 비교 그림은 `../scripts/tsdf_slice_check.png`에 남긴다.

- **렌더링으로 풀기 (부록 A):** 광선 데모는 손으로 정한 예시 장면·색·밀도이며 실제 데이터나 학습된 신경망이 아니다. NeRF는 같은 간격의 점, Gaussian Splatting은 타원체 셋을 같은 앞에서부터 색 쌓기 식으로 비교한다. 점 개수와 앞 물체 불투명도를 바꾸고, ‘고치기’로 사진 픽셀과의 색 차이를 수치 미분해 불투명도를 한 걸음씩 학습한다. 사진 픽셀도 숨은 예시 불투명도로 만든 색이다.
  - Mildenhall 외, **NeRF: Representing Scenes as Neural Radiance Fields for View Synthesis**, ECCV 2020: [공식 프로젝트](https://www.matthewtancik.com/nerf).
  - Kerbl 외, **3D Gaussian Splatting for Real-Time Radiance Field Rendering**, SIGGRAPH 2023: [공식 프로젝트](https://repo-sam.inria.fr/fungraph/3d-gaussian-splatting/).
  - Wang 외, **NeuS: Learning Neural Implicit Surfaces by Volume Rendering for Multi-view Reconstruction**, NeurIPS 2021: [공식 프로젝트](https://lingjie0206.github.io/papers/NeuS/).

- **최근 연구 (부록 B):** 2D 데모는 손으로 정한 예시 값이며, 실제 데이터가 아닌 **2D로 줄인 그림**이다. 원문의 식으로 GGGS의 평균·이산 중간값·연속 중간값 깊이를 비교하고, Gaussian Wrapping의 카메라 광선 기반 빈 정도에서 닫힌 선을 꺼낸다. 두 삼각망 데모는 직접 구현한 Bowyer–Watson Delaunay를 공유한다. MeshSplatting은 삼각형 창 함수·불투명도 하한과, 쌍대 Voronoi 변이 수프와 교차하는 변만 고르는 제한 Delaunay를 보여 준다. 새 꼭짓점을 만들지 않는다.
  - **논문 그림:** `data/meshsplatting_fig3.png`는 Held 외, MeshSplatting (CVPR 2026, [arXiv 2512.06818](https://arxiv.org/abs/2512.06818)) 그림 3을 잘라낸 이미지이며, 라이선스는 CC BY 4.0이다. `data/gaussian_wrapping_fig1.png`, `data/gaussian_wrapping_fig3.png`는 Gomez 외, From Blobs to Spokes (ECCV 2026, arXiv 2604.07337) 그림 1·3을 잘라낸 이미지다. arXiv 기본 라이선스(재배포 허락 없음)이며, 저장소 소유자의 결정으로 출처를 밝혀 교육용으로 인용했다. 저작권자가 요청하면 삭제한다.
  - Gaussian Wrapping의 후보 점은 타원체마다 중심과 법선 방향 바깥 점 2개다. 기존 방식은 2D에서는 중심 + 3σ 상자 모서리 4개 = 5개, **3D에서는 중심 + 모서리 8개 = 9개**다.
  - GGGS의 지수에는 ½가 없고, Gaussian Wrapping에는 ½가 있다. MeshSplatting의 부호 거리는 원문 식 (1)의 바깥 법선 거리 최댓값으로 계산한다. 식은 `NOTES_p9.md`와 제공된 원문 텍스트를 대조했다.
  - Baowen Zhang, Chenxing Jiang, Heng Li, Shaojie Shen, Ping Tan, **GGGS: Geometry-Grounded Gaussian Splatting**, arXiv 2601.17835 (공개 2026-01-25, 수정 2026-01-27), 학회 미확인: [논문](https://arxiv.org/abs/2601.17835). 더 정확히 렌더링한 깊이를 TSDF로 합쳐 Marching Cubes로 면을 뽑는다.
  - Diego Gomez\*, Antoine Guédon\*, Nissim Maruani, Bingchen Gong, Maks Ovsjanikov (\*공동 1저자), **Gaussian Wrapping — From Blobs to Spokes: High-Fidelity Surface Reconstruction via Oriented Gaussians**, arXiv [2604.07337](https://arxiv.org/abs/2604.07337) (공개 2026-04-08), ECCV 2026: [공식 프로젝트](https://diego1401.github.io/BlobsToSpokesWebsite/). 타원체마다 법선을 배우고, Marching Tetrahedra 등으로 닫힌 면을 꺼낸다.
  - Jan Held, Sanghyun Son, Renaud Vandeghen, Daniel Rebain, Matheus Gadelha, Yi Zhou, Anthony Cioppa, Ming C. Lin, Marc Van Droogenbroeck, Andrea Tagliasacchi, **MeshSplatting: Differentiable Rendering with Opaque Meshes**, arXiv [2512.06818](https://arxiv.org/abs/2512.06818) (공개 2025-12-07), CVPR 2026: [학회 논문](https://openaccess.thecvf.com/content/CVPR2026/html/Held_MeshSplatting_Differentiable_Rendering_with_Opaque_Meshes_CVPR_2026_paper.html). 연결된 삼각형 면의 모양과 색을 렌더링으로 함께 학습한다.
  - Miller 외, **Objects as Volumes: A Stochastic Geometry View of Opaque Solids**, CVPR 2024: [학회 논문](https://openaccess.thecvf.com/content/CVPR2024/html/Miller_Objects_as_Volumes_A_Stochastic_Geometry_View_of_Opaque_Solids_CVPR_2024_paper.html). 빈 정도에서 방향별 감쇠를 구하는 공통 이론.
  - Held 외, **Triangle Splatting for Real-Time Radiance Field Rendering**, arXiv 2505.19175: [논문](https://arxiv.org/abs/2505.19175). 화면 삼각형의 부호 거리로 창 함수를 정의한다.

## 외부 코드

- Numeric Javascript 1.2.6 (Sébastien Loisel, MIT License): `data/numeric.min.js`. SVD 계산에 쓴다.
- KaTeX 0.16.11 (Khan Academy, MIT License): `data/katex/`.
