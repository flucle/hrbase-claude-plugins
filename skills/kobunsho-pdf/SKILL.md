---
name: kobunsho-pdf
description: 公文書の ZIP を HRbase で PDF に変換し、指定したリネーム名で保存する。公文書、ZIP、PDF 変換、リネームを頼まれたときに使う。
---

指定フォルダの公文書 ZIP を PDF にし、指定フォルダへ保存する。

1. ZIP ごとに、ファイル名とバイトサイズを `prepare_kobunsho_upload` に渡す。
2. 返った `id` と `token` で、`POST /api/ai_assistants/attachment_upload_grants/:id/attachment_files` へ ZIP を送る。`token` はヘッダー `HRbase-Attachment-Upload-Token` に付ける。
3. 返った attachment file の ID と、リネーム名を含む Question を `convert_kobunsho_to_pdf` に渡す。
4. 返った `downloadUrl` から PDF を取得し、指定フォルダへ `fileName` で保存する。
