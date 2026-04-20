---
name: storage-gcs
description: Google Cloud Storage — 영상/폰트 업다운로드 + Signed URL 생성
user-invocable: false
paths: short-video-maker/src/storage/GoogleCloudStorageService.ts
---

## 용도
영상 파일 업로드/다운로드, Signed URL 생성 (24시간), 폰트 파일 관리.
모든 프로젝트(Books, News, Poetry)에서 공통 사용하는 스토리지 레이어.

## 핵심 파일
- `src/storage/GoogleCloudStorageService.ts` — 메인 클래스 (싱글턴)

## 의존성
- `@google-cloud/storage` — GCS SDK
- 환경변수: `GCS_BUCKET_NAME` (기본값: `YOUR_GCS_BUCKET`)
- GCP 기본 인증 (Application Default Credentials)

## API/인터페이스
```typescript
class GoogleCloudStorageService {
  // 영상 업로드/다운로드
  async uploadVideo(localPath: string, remotePath: string): Promise<string>;
  async downloadVideo(remotePath: string, localPath: string): Promise<void>;

  // Signed URL (24시간 유효)
  async generateSignedUrl(remotePath: string): Promise<string>;

  // 폰트 관리
  async downloadFonts(fontNames: string[], localDir: string): Promise<void>;
  async uploadFonts(localDir: string, fontNames: string[]): Promise<void>;
}
```

## 다른 프로젝트에 이식하기
1. `GoogleCloudStorageService.ts` 복사
2. `npm install @google-cloud/storage`
3. GCP 프로젝트 생성 + 서비스 계정 키 설정 (또는 `gcloud auth application-default login`)
4. 환경변수 `GCS_BUCKET_NAME` 설정
5. 버킷 생성: `gsutil mb gs://your-bucket-name`

## 주의사항
- Signed URL은 24시간 후 만료 — 장기 보관용이 아님
- 버킷은 리전별 요금 상이 (asia-northeast3 서울 권장)
- 대용량 영상은 resumable upload 사용됨
- 폰트 파일은 Cloud Run 배포 시 로컬에 없으므로 GCS에서 다운로드 필요
- 서비스 계정에 `storage.objectAdmin` 역할 필요
