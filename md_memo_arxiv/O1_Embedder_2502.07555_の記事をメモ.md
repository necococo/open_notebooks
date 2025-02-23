---
marp: true
---

https://zenn.dev/knowledgesense/articles/3ecad11999fea3　より。


今回は、Embeddingの過程で深い思考を行うモデル「O1 Embedder」について紹介します。


サマリー
EmbeddingはRAGのシステムにも欠かせないもので、Embeddingの性能がそのままRAGの性能に直結すると言っても過言ではありません。

O1 Embedderは、EmbeddingとOpenAIのo1モデルのような深い思考をかけ合わせたモデルです。この特徴によって、特に複雑なクエリに対するEmbeddingの性能を向上させることに成功しました。

OpenAIのo1モデルに代表するような深い思考を実現するモデルは、クエリを元に様々な観点で考えを洗い出し深く考えることで精度を高めています。Embeddingでも同じことを実現したのが「O1 Embedder」です。(特に論文内で明記されていませんが、O1はopenaiのo1モデルからそのまま取ったものだと思います。)

問題意識
手法


O1 Embedderは、LLMとしての機能とEmbeddingの機能を併せ持つLLMです。RAGにおけるドキュメントの保管の際には、通常通りEmbedding機能のみを利用してEmbeddingを行います。クエリによる検索の際には、ユーザーがどういう意図で質問したかを分析する深い思考を行い、その結果をEmbeddingすることで関連するドキュメントを検索します。

事前学習
O1 Embedderは既存のLLMをFTすることで、作成していきます。もととなるLLMに制約は基本的にありませんが、基本的にはLlama-2-7Bをベースとしています。

1. 思考の学習
入力されたクエリを元に、どのように思考を拡張すればよいか、を学習していきます。クエリと想定される思考の出力ペアを用いた教師あり学習を利用することで、思考の生成を学習していきます。

2. Embeddingの学習
入力されたクエリを元に、適切な文書を検索できるように対照学習を利用して学習します。クエリだけではなく、思考の結果も付加したうえでの検索性能の向上を目指して学習していきます。

3. 思考とEmbeddingの共同学習
最後に、クエリを元に深い思考をしてEmbedding作成するプロセス全体で学習を行います。

利用方法
1．クエリを深い思考を用いて拡張。

例えば、「東京タワーの設計者は他にどんな建物を設計しましたか？」というクエリに対して、『東京タワーの設計者は内藤多仲（ないとう たちゅう）です。彼は「塔博士」とも呼ばれ...』といった拡張を行います。

拡張された文章全体を用いて、Embeddingを生成
生成されたEmbeddingを利用してドキュメントを検索
「O1 Embedder」の特徴は、クエリに対する深い思考とEmbeddingをまとめて同じモデルを使用して行っている点にあります。これにより、より高いベクトル化の精度を獲得しています。

成果


Embeddingを用いた検索結果の比較表です。w/o Tと表記されているものは、深い思考を行わなかった場合のO1 Embedderの性能を指しています。「O1 Embedder」は既存のEmbeddingモデルと比較して平均で2%程度、性能が改善しています。特に思考の有無に着目すると、思考の無い状態ではそれまでの手法と大差ないのに対して、深い思考を行うことで性能が改善していることが確認できます。



通常のEmbeddingと深い思考を加えたEmbeddingによる性能の差についてまとめたものです。比較対象に上がっている、RepLLaMAは深い思考を追加してもEmbeddingの性能がほとんど変わっていないのに対して、O1 Embedderは2%程度の性能の向上が確認できております。

まとめ
OpenAIのo1モデルから始まり、GeminiやDeepSeekなどのモデルも深い思考という文脈での開発が進んでおり、ますますLLMの躍進が見られています。今回紹介した「O1 Embedder」は、その考え方をEmbeddingに拡張することで一定の成果を上げています。特に、他のモデルではLLMによるクエリの拡張ではあまり効果が上がらないというのは非常に面白いポイントだと思います。LLMとEmbeddingモデル側の相性が重要になり、それらが統合されているO1 Embedderは当然相性が良く性能が高くなるのだと言えます。

一方で、専門性の高い分野での性能は改善の余地がありそうです。例えばFiQA(金融関連)という比較軸においては、最高性能を実現できていません。いくら深く考える能力があってもそもそも知らなければ精度が上がらないことを示しています。専門性の高い領域では特に、ドメインに合わせたFTが重要になりそうです。


