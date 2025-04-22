# SMARTBOX

この JavaScript モジュールは、入力ボックスに対してキーワードの候補をサジェストする機能を提供します。ローカルの TSV ファイルから取得したデータや、必要に応じて外部 API から取得したデータに基づいてキーワードをサジェストします。サジェストされたキーワードは、入力ボックスの下にドロップダウンで表示され、ユーザーが選択できるようになります。

## デモについて

以下 URL でプロパティやオプションを確認できます。
https://pubcasefinder.github.io/pubcasefinder_tools/smartbox/

### ローカルでの使用手順

1. 以下のコマンドをターミナルで入力してディレクトリを移動してください。

```sh
cd smartbox
```

#### smartBox の引数

- `smart-box-id`: 入力ボックス要素の ID。必須です。
- `data-path`: TSV ファイルの URL。必須です。
- `placeholder`: 入力ボックスのプレースホルダーテキスト。省略可能です。
- `options`: 追加設定を指定する JSON オブジェクト。省略可能です。
  - `include_no_match`: キーワードが見つからない場合にキーワード自体の選択欄を含めるかどうか（デフォルト: false）
  - `max_results`: サジェストボックスに表示する最大の候補数

2. もしかして検索（API）をローカルで試したい場合は、API サーバーを起動してください。デモ用に作成した（server.js）は Node.js と Express を使用しています。簡易的な API であるため、実際のプロジェクトで使用する場合は適切な API を用意してください。

```sh
npm install
```

```sh
node demo/js/server.js
```

3. VSCode の拡張機能である Live Server を起動してください。その後、http://127.0.0.1:5500/smartbox/ を開いてください。

## 他プロジェクトでの使い方

デモページで表示される以下のようなコードを html ファイルに貼り付けてください。

```html
<link
  rel="stylesheet"
  href="https://cdn.jsdelivr.net/gh/PubCaseFinder/pubcasefinder_tools@main/smartbox/style.css"
/>

<smart-box
  smartbox-id="12"
  data-path="https://raw.githubusercontent.com/PubCaseFinder/pubcasefinder_tools/main/smartbox/tsv/nando_sample.tsv"
  placeholder="Search..."
  options='{"include_no_match":false}'
></smart-box>

<script
  type="module"
  src="https://cdn.jsdelivr.net/gh/PubCaseFinder/pubcasefinder_tools@main/smartbox/smartbox.js"
></script>

<script type="module">
  document.addEventListener('selectedSmartBoxLabel', (e) =>
    console.log(e.detail.smartBoxId, e.detail.labelInfo)
  );
</script>
```

### インターネットに繋がらないようなプロジェクトの場合は、smartbox.js と style.css をコピーしてプロジェクトのディレクトリに配置してください。

#### HTML

html ファイルに以下のようなコードを記述してください。また、id は任意のものを指定してください。

```html
<link rel="stylesheet" href="your_project_directory/style.css" />

<smart-box
  smart-box-id="sample"
  data-path="https://raw.githubusercontent.com/PubCaseFinder/pubcasefinder_tools/main/smartbox/tsv/nando_sample.tsv"
  placeholder="Search..."
  options='{"include_no_match":false}'
></smart-box>

<script type="module" src="your_project_directory/smartbox.js"></script>
```

#### CSS

style.css に含まれるコードをコピーしてご利用のプロジェクトの CSS ファイルに追加してください。
css は scss で管理しています。Smart Text Box 共通部分に関わる修正をする場合は scss 自体も変更・更新してください。

#### JavaScript

smartbox.js ファイルをコピーして追加してください。利用したい js ファイルで以下のように呼び出してください。パスは適宜変更してください。

```javascript
// カスタムイベントのリスナー（インプットボックスのIDと選択したラベル情報のオブジェクトを取得し、Consoleに表示する例）
<script type="module">
  document.addEventListener('selectedSmartBoxLabel', e =>
  console.log(e.detail.smartBoxId, e.detail.labelInfo));
</script>
```

### TSV

TSV ファイルは以下のような構成にしてください。

```tsv
id label_en synonym_en label_ja synonym_ja
```

## smartbox.js の詳細

### 概要

この JavaScript モジュールは、入力ボックスに対してキーワードの候補をサジェストする機能を提供します。ローカルの TSV ファイルから取得したデータや、必要に応じて外部 API から取得したデータに基づいてキーワードをサジェストします。サジェストされたキーワードは、入力ボックスの下にドロップダウンで表示され、ユーザーが選択できるようになります。

#### 関数の説明

##### smartBox

目的: ユーザーの入力に基づいてキーワードの候補をサジェストします。

パラメータ:
input_box_id (string): 入力ボックスの要素 ID。
data_path (string): キーワードデータを含む TSV ファイルのパス。
options (object, 任意):
api_url (string): 追加のキーワード候補を取得するための API の URL。
include_no_match (boolean): キーワード自体の選択欄をサジェストボックスに含めるかどうか。
max_results (number): サジェストボックスに表示する最大の候補数。

##### 機能

###### 初期化:

- 指定された TSV ファイルからキーワードデータを取得します。
- 必要に応じて API から追加の候補を取得します。
- ユーザーの入力やインタラクションに対するイベントリスナーを設定します。

###### イベントリスナー:

- input: ユーザーの入力に基づいてキーワードの候補を表示します。
- keydown: 提案されたキーワードのドロップダウン内でのキーボードナビゲーションを処理します。
- compositionstart および compositionend: 複雑な入力方法を持つ言語の入力構成を処理します。
- focus: 入力ボックスがフォーカスを取得したときに検索を再トリガーします。
- click (outside): サジェストボックスの外側をクリックしたときにサジェストボックスを非表示にします。

###### キーワードサジェストの処理:

- 入力値を正規化し、ローカルデータから一致するキーワードを検索します。
- 一致するキーワードが見つからない場合、API から追加の候補を取得します。
- 提案されたキーワードをドロップダウン形式で表示し、ユーザーが選択できるようにします。

###### キーワードのハイライト:

- 提案されたキーワードの中で入力に一致する部分をハイライト（太字）表示します。

###### 選択の更新:

- キーボードやマウス操作によって選択されたキーワードの項目を更新します。

#### 使用例

デモページでの出力を参考にオプションを追加してください。

#### もしかして検索（API）のレスポンス

もしかして検索の API は、以下のような形式でレスポンスを返すことを想定しています。

```javascript
[
  {
    id: 'D1',
    label_en: 'Disease A',
    synonym_en: 'Alternative name for Disease A',
    label_ja: '病気A',
    synonym_ja: '病気Aの別名',
  },
  {
    id: 'D2',
    label_en: 'Disease B',
    synonym_en: 'Alternative name for Disease B',
    label_ja: '病気B',
    synonym_ja: '病気Bの別名',
  },
  {
    id: 'D3',
    label_en: 'Disease C',
    synonym_en: 'Alternative name for Disease C',
    label_ja: '病気C',
    synonym_ja: '病気Cの別名',
  },
];
```
