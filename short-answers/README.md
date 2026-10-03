# 短い答えをJevに読ませた検証

noteの記事「[【AI×教育】小テストの採点を、AIに手伝ってもらえるか：判断を数値で返すAI「Jev」で試した](https://note.com/yamnor/n/n8c396ace9778)」の補足資料です。

## 検証の概要

- **題材**：食塩の溶解についての一問一答6問（短い自由記述）
- **答え**：各問に5種類（正答、表記の揺れ、否定を含む答え、もっともらしい誤答、答えになっていない答え）、計30の検証用に作った答え
- **尋ねたこと**：一つの答えにつき、次の二つを一回の送信で尋ねた。
  - 模範解答と同じ内容か（Noul）
  - どの考えに最も近いか（Choice）
- **比べた条件**：Choiceの選択肢に「該当なし（どの考えにも当たらない）」を入れるか入れないか
- **回数**：30の答え×2条件×3回＝180回
- **モデルと日付**：jev-1.13.0、2026年10月2日
- **費用**：入力は計138,966トークン。公式料金（入力100万トークンあたり0.042ドル、出力は無料）で約0.0058ドル。

## 主な結果

- 「同じ内容か」で、期待と逆の判定は180回のどこにもなかった。期待と違ったのは4つの答えで、どれも「不確か」（0.20以上0.80未満）だった。
- 3回の測定で、判定と選ばれた考えは、両条件とも全30の答えで変わらなかった。
- 「どの考えに近いか」は、該当なしを入れた条件で、3回とも30の答えすべてが期待どおりだった。
- 該当なしを入れない条件では、答えになっていない6つの答えのうち4つが「正しい考え」に分類された。うち2つは確率0.97〜0.99だった。

詳しくは [results/findings.txt](results/findings.txt) を見てください。

## ファイル

- [protocol.txt](protocol.txt)：送信前に固定した仕様、事前の見立て、変更履歴
- [questions.md](questions.md)：送信したプロンプトの全文と、問ごとの選択肢
- data/
  - [dataset.json](data/dataset.json)：6問の問題、模範解答、考えの選択肢、30の答え
  - [labels.json](data/labels.json)：期待する判定。Jevには送っていない。
  - [option_letters.json](data/option_letters.json)：Choiceの選択肢に付けた記号（A〜D）と考えの対応
- examples/
  - [request_response_example.json](examples/request_response_example.json)：実際に送った要求と、返ってきた応答の一例
- results/
  - [summary.json](results/summary.json)：集計
  - [details.csv](results/details.csv)：180回の送信の一つずつの結果
  - [findings.txt](results/findings.txt)：結果の読み解き

## details.csv の列

- answer_id、question_id、kind：答えのID、問のID、答えの種類
- condition：with_none は該当なしを入れた条件、without_none は入れない条件
- repeat：何回目の測定か（1〜3）
- noul：「同じ内容か」の「はい」の確率
- same_meaning：判定（match＝一致、mismatch＝不一致、uncertain＝不確か）
- expected_same_meaning：期待した判定（True＝一致を期待）
- idea、idea_probability、idea_confidence：選ばれた考え、その確率、Jevが返した確信度
- expected_idea：期待した考え。該当なしを入れない条件の「答えになっていない答え」には正しい選択肢がないので空欄。

## 注記

- 答えはAIに下書きさせ、著者が測定前に確認した。labels.json の labeled_by 欄は、確認前に固定した記述（「著者の確認待ち」）のまま残している。送信前に固定したファイルを書き換えないためである。
- protocol.txt と results/findings.txt は、公開にあたり題名と「人工解答」という言葉を整えた。仕様と結果の中身は変えていない。data/ のファイルは送信前に固定したままなので、dataset.json の description 欄などに当時の記述（「第1回」「人工の解答」）が残っている。
- 最初の測定は、集計プログラムの検証の誤りで18回目に止めた。Jevは選択肢の確率を小数第2位に丸めて返すため、合計が0.99になる応答を不正と判定していた。条件を変えずに測り直し、その結果だけを集計した。経緯は protocol.txt の変更履歴にある。
- 実行コードと、180回分の生の応答は公開していない。
