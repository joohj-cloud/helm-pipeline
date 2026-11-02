# helm-pipeline

Helm 스테이지 E2E 테스트용 차트(helm-e2e). 차트는 저장소 최상위에 있으므로 Deploy의 차트 경로는 비워 둔다.

리소스(이름은 릴리스 이름 기준): ServiceAccount, ConfigMap(-config), Secret(-secret), Deployment(nginx), Service, hook Job(조건부)

| 값 | 기본 | 용도 |
|----|------|------|
| message | chart-default | ConfigMap message·index.html, Values 적용 확인 |
| replicaCount | 1 | Pod 수 |
| env | LOG_LEVEL: info | 컨테이너 환경 변수(맵이라 Values 파일끼리 키가 합쳐짐) |
| features | [] | ConfigMap features(목록이라 마지막 값으로 통째 교체, Values 파일로만 입력) |
| image.tag | stable | 없는 태그로 이미지 실패 |
| readyDelaySeconds | 0 | Pod 준비 지연(완료 대기·Timeout) |
| hook.sleepSeconds | 0 | 0보다 크면 pre-install·pre-upgrade hook이 그만큼 대기 |
| hook.fail | false | true면 hook 실패 |
| failMessage | "" | 값이 있으면 렌더링 실패 |
| secret.E2E_SECRET | e2e-dummy-secret-value | 출력 노출 검색용 더미 값 |

Values 파일

| 파일 | 내용 |
|------|------|
| env/values-a.yaml | message: from-a, replicaCount: 2, env LOG_LEVEL: debug·FROM_A, features: a-1, a-2 |
| env/values-b.yaml | message: from-b, env LOG_LEVEL: warn·FROM_B, features: b-1 |

a → b 순서면 message from-b, replicas 2, env는 FROM_A·FROM_B 모두 있고 LOG_LEVEL warn, features b-1만 남는다.

참고: hook Job은 helm 관리 리소스가 아니라 Delete 후에도 남는다(다음 hook 생성 때 교체).
