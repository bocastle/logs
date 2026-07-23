# S3 multipart upload 완료 실패를 복구하는 법 정리

## 핵심 요약

- Multipart upload는 part별 재시도를 가능하게 하지만 완료 요청의 응답 유실까지 자동으로 판별해 주지는 않는다.
- 서버가 upload ID와 part number·ETag 목록, 최종 object key와 checksum을 내구성 있게 저장한다.
- Complete 결과가 불명확하면 먼저 object 존재와 식별 정보를 확인하고, 미완료 상태일 때만 ListParts와 재완료를 시도한다.

## 개념 설명

S3 multipart upload는 파일을 여러 part로 나눠 업로드한 뒤 ETag 목록으로 완료 요청을 보낸다. 네트워크가 끊겨 client가 완료 응답을 받지 못했어도 S3에서는 object 생성이 이미 성공했을 수 있어 단순 재시도만으로 판단하면 안 된다.

서버는 upload ID와 part 목록을 상태 저장소에 기록한다. 결과가 불명확하면 고유 object key에 `HEAD`를 보내 예상 크기, checksum이나 metadata가 맞는지 먼저 본다. Object가 없다면 upload ID로 `ListParts`를 조회해 완료 가능한지 판단한다. 이미 완료된 upload ID는 더 이상 multipart 상태로 조회되지 않을 수 있다는 점도 정상 분기로 처리한다.

## 예시

```text
CreateMultipartUpload -> upload_id 저장
UploadPart(part=1..n) -> ETag 저장
Complete 실패 -> ListParts(upload_id)
all parts ok -> retry Complete
else -> AbortMultipartUpload
```

실제 복구 순서는 object 확인을 앞에 둔다. 같은 key를 다른 업로드가 덮어쓸 수 있다면 `HEAD`만으로 자신의 결과인지 알 수 없으므로 업로드별 고유 key나 metadata·checksum을 사용한다.

## 면접 답변 예시

> Multipart complete 응답이 끊겨도 S3에서는 object가 이미 생성됐을 수 있습니다. 그래서 upload ID와 part ETag, 고유 object key와 checksum을 서버 상태로 저장하겠습니다. 결과가 불명확하면 먼저 `HEAD`로 자신의 object가 완성됐는지 확인하고, 없다면 `ListParts`로 미완료 upload 상태를 확인해 complete를 재시도합니다. 끝내 복구할 수 없는 upload는 abort하고 오래된 미완료 part는 lifecycle rule로 정리합니다.

## 장점

- 실제로 미완료라면 이미 성공한 part를 다시 전송하지 않고 complete를 재시도할 수 있다.
- Part별 상태를 저장하면 대용량 업로드 실패 범위를 좁힐 수 있다.

## 단점

- 완료 여부가 불명확한 상태에서 같은 key로 새 업로드를 시작하면 결과 object를 잘못 판별할 수 있다.
- 미완료 multipart가 쌓이면 보이지 않는 part 저장 비용이 계속 증가한다.

## 주의사항 / 실무 팁

- Upload별 고유 key 또는 metadata와 checksum으로 완료된 object의 소유를 확인한다.
- Lifecycle rule로 오래된 incomplete multipart를 정리하고 abort 수와 미완료 byte를 관찰한다.
