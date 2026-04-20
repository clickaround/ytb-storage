---
name: storage-youtube-upload
description: YouTube 멀티채널 업로드 — OAuth + Secret Manager 기반 자동 업로드/댓글
user-invocable: false
paths: short-video-maker/src/youtube-upload/**
---

## 용도
멀티채널 YouTube 영상 업로드, 첫 댓글 자동 게시. 채널별 OAuth 토큰을
Secret Manager에서 관리하며, 채널 이름으로 자동 토큰 선택.

## 핵심 파일
- `src/youtube-upload/services/YouTubeUploader.ts` — 메인 업로드 클래스
- `src/youtube-upload/services/YouTubeSecretManager.ts` — Secret Manager 토큰 관리
- `src/youtube-upload/services/YouTubeAuthManager.ts` — OAuth 인증 흐름

## 의존성
- `googleapis` — YouTube Data API v3
- `@google-cloud/secret-manager` — GCP Secret Manager
- 환경변수: `YOUTUBE_CLIENT_SECRET_PATH`, `YOUTUBE_DATA` (Secret Manager, base64 tar.gz)

## API/인터페이스
```typescript
class YouTubeUploader {
  async uploadVideo(
    videoId: string,
    channelName: string,   // "main-channel" | "sub-channel-1" | "poetry-channel" 등
    metadata: { title: string; description: string; tags: string[] },
    isPublic: boolean
  ): Promise<string>; // YouTube video ID

  async postComment(
    videoId: string,
    channelName: string,
    text: string
  ): Promise<void>;

  async isChannelAuthenticated(channelName: string): Promise<boolean>;
}
```
- 채널 목록: `main-channel`, `sub-channel-1`, `news-channel-1`, `news-channel-2`, `poetry-channel`

## 다른 프로젝트에 이식하기
1. `src/youtube-upload/` 폴더 전체 복사
2. `npm install googleapis @google-cloud/secret-manager`
3. Google Cloud Console에서 OAuth 2.0 클라이언트 생성 (Desktop App)
4. 채널별 OAuth 토큰 발급 (`YouTubeAuthManager`로 인증 흐름 실행)
5. 토큰을 base64 tar.gz로 압축 → Secret Manager에 `YOUTUBE_DATA`로 저장
6. `YOUTUBE_CLIENT_SECRET_PATH`에 OAuth 클라이언트 시크릿 JSON 경로 설정

## 주의사항
- **OAuth 토큰 만료 시 재인증 필요** — `force-ssl` scope 포함 필수
- Secret Manager 버전 누적 방지 — 업데이트 시 이전 버전 삭제 권장
- YouTube API 일일 쿼터 제한 (10,000 units/day, 업로드 1건 = 1,600 units)
- 비공개(unlisted) 업로드 후 수동 공개 전환 가능
- 첫 댓글은 업로드 직후 자동 게시됨 (SEO/해시태그 용도)
