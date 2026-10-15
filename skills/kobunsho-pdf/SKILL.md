---
name: kobunsho-pdf
description: 公文書の ZIP を添付されたとき、HRbase の MCP ツールで PDF に変換し、ファイル名を付け替えてダウンロードリンクを表示する。公文書、ZIP、PDF 変換、リネームの依頼で使う。
---

添付した公文書の ZIP ファイルを、HRbase の MCP ツール `HRbase` を使って PDF に変換し、ファイル名を付け替えてください。
複数の ZIP があるときは、ZIP ごとに次の手順を実行してください。

【手順 1】アップロード許可を発行する
- シェルで ZIP のファイル名とバイト数を取得してください。バイト数は `stat -f%z <ファイル>`（macOS）または `stat -c%s <ファイル>`（Linux）で取得します。推測や計算はしないでください。
- MCP ツール `prepare_kobunsho_upload` を、`files: [{ "fileName": "<ZIP のファイル名>", "byteSize": <バイト数> }]` で呼んでください。
- 返ってきた `id` と `token` を控えてください。token は一度きり、1 時間有効で、1 回使うと使えなくなります。この会話の外には出さないでください。

【手順 2】ZIP を HRbase にアップロードする
- 次の curl で、ZIP のバイト列そのものを POST してください。JSON や Base64 にはしないでください。
  curl -sS -X POST "https://qa-hrbase.link/api/ai_assistants/attachment_upload_grants/<id>/attachment_files" \
    -H "HRbase-Attachment-Upload-Token: <token>" \
    -H "Content-Type: application/octet-stream" \
    --data-binary @"<ZIP のパス>" \
    -w "\nHTTP %{http_code}\n"
- Content-Length は、手順 1 で申告した byteSize と一致する必要があります。`--data-binary @ファイル` なら自動で一致します。
- 201 が返れば成功です。応答の `attachment_file.id` を控えてください。
- 失敗した場合は、同じ id と token を再送せず、手順 1 からやり直してください。token は使い切りです。HTTP ステータスとエラー内容を利用者に報告してください。

【手順 3】PDF に変換し、ダウンロードリンクを回答に表示する
- MCP ツール `convert_kobunsho_to_pdf` を、`attachmentFileIds: ["<attachment_file.id>"]` と、次の `question` で呼んでください。
  question: 会社名_申請書名_従業員氏名
- ファイル名を指定しない場合は、`question` を「開始して」にしてください。
- 返ってきた `answer` を、そのまま利用者に見せてください。
- ファイルのダウンロードやフォルダへの保存は行わないでください。
- `files[]` の各 `downloadUrl`（署名付き URL、有効期限 60 分）を、`fileName` にリンクを付けた Markdown 形式 `[<fileName>](<downloadUrl>)` の番号付きリストにして、回答に表示してください。URL は加工、省略、改変をせず、そのまま使ってください。
- リンクの有効期限が 60 分であることを、リストの前後のどちらかに一言添えてください。

【注意】
- 各ツールのエラー本文（isError）や HTTP ステータスは、省かずに利用者へそのまま報告してください。
- 変換に失敗した旨の回答が返り、`files` が空のときは、答えをそのまま報告し、再実行する前に利用者に確認してください。
