# 静岡の気候はどう変わったか — 1940-2025年 時系列分析

Open Data Experimental Lab の実験成果物です。
気象庁の公開する一次データだけを使い、第三者が同じ手順で再現できる形にしています。

> **公開版について**：このGitHub Pages版では、レポートはWord版をHTMLに変換した `report.html` として収録し、図5点は `index.html` と `dashboard.html` のHTML内に埋め込んでいます（下記のディレクトリ構成は、再現用プロジェクト側の構成を示したものです）。

## 結論（先に3行）

1. 静岡の年平均気温は 1940-2025年で **10年あたり +0.26℃**（100年あたり +2.6℃）上昇した。Mann-Kendall 検定 p<0.001。
2. **「冬の方が夏より速く暖まる」という仮説は支持されなかった**（差の95%信頼区間が0を含む）。月別では3月が最も速い。
3. **年降水量・年最大日降水量に長期トレンドは検出されなかった**。「大雨が増えている」とは、このデータからは言えない。

## 実験テーマ

テーマ9「時系列変化の分析」。対象は静岡（気象庁 47656）。

### 仮説

| ID | 仮説 | 判定 |
|---|---|---|
| H1 | 静岡の年平均気温は1940-2025年に有意に上昇している | **支持** |
| H2 | 冬（12・1・2月）の昇温は夏（6・7・8月）より大きい | **不支持** |
| H3 | 年降水量・年最大日降水量には長期トレンドがない | **支持** |

## ディレクトリ構成

```
experiments/shizuoka_climate/
├── README.md                  ← 本ファイル（再現手順）
├── SOURCES.md                 ← 出典・ライセンス・取得条件
├── verification.md            ← 独立検証の結果と方法
├── data/
│   ├── raw/                   ← 気象庁HTML（無加工）＋ fetch_log.json ＋ sources.json
│   └── processed/             ← 加工済みCSV ＋ data_dictionary.md
├── src/
│   ├── fetch_data.py          ← STEP2 データ取得
│   ├── parse_data.py          ← STEP2-3 解析・CSV化
│   ├── stats_tools.py         ← Mann-Kendall / Sen / Pettitt など
│   ├── analyze.py             ← STEP4 分析 → output/results.json
│   ├── verify.py              ← STEP5 独立検証 → output/verification.json
│   ├── make_figures.py        ← STEP6 図5点
│   ├── make_dashboard.py      ← STEP6 ダッシュボードHTML
│   └── make_report.py         ← STEP6 レポート(docx)
└── output/
    ├── results.json           ← 全数値
    ├── verification.json      ← 検証結果
    ├── figures/*.png          ← 図5点
    ├── dashboard.html         ← 単体で開けるダッシュボード
    └── 静岡の気候変動_1940-2025.docx
```

## 必要環境

- Python 3.12
- 追加パッケージ: `requests`, `pandas`, `numpy`, `lxml`, `python-docx`, `matplotlib`
- フォント: `C:\Windows\Fonts\YuGothR.ttc`（Yu Gothic）
- ネットワーク接続（初回のデータ取得のみ）

```powershell
pip install requests pandas numpy lxml python-docx matplotlib
```

## 再現手順

プロジェクトルート（`experiments/shizuoka_climate`）で、上から順に実行します。

```powershell
# 1. データ取得（data/raw/ に保存。2回目以降はスキップ）
python src\fetch_data.py

# 2. 解析してCSV化
python src\parse_data.py

# 3. 分析（output/results.json を生成）
python src\analyze.py

# 4. 独立検証（output/verification.json を生成）
python src\verify.py

# 5. 図とダッシュボード
python src\make_figures.py
python src\make_dashboard.py

# 6. レポート(docx)
python src\make_report.py
```

- 生データを取り直す場合のみ `python src\fetch_data.py --force`。
- 乱数を使う検証（ブートストラップ・並べ替え検定）は seed 固定。同じ結果になります。
- 実行順を変えると `results.json` が無くて失敗します。必ず 1→6 の順で。

## 主な結果

| 指標 | 値 | 検定 |
|---|---|---|
| 年平均気温の傾向（1940-2025） | +0.261℃/10年（95%CI +0.223〜+0.296） | Mann-Kendall p<0.001 |
| 100年あたり | +2.61℃ | - |
| 1941-1970 → 1996-2025 | 15.73℃ → 17.16℃（+1.43℃） | - |
| 最高年 | 2024年 18.7℃（平年比 +1.8℃） | - |
| 最低年 | 1945年 14.6℃ | - |
| 3月の昇温 | +0.368℃/10年 | p<0.001 |
| 7月の昇温 | +0.186℃/10年 | p<0.001 |
| 冬 − 夏 の差 | +0.034℃/10年（95%CI −0.051〜+0.116） | 有意でない |
| 年降水量 | +12.0mm/10年 | p=0.48（有意でない） |
| 年最大日降水量 | +0.71mm/10年 | p=0.83（有意でない） |
| 年最大1時間降水量 | +1.48mm/10年 | p=0.052（境界） |
| 変化点（トレンド除去後） | 2017年 | p=0.33（段差なし） |

## 検証（STEP5）

`output/verification.json` の6項目すべて合格。

1. 生HTMLを pandas と別方法（正規表現）で再解析 → 最大差 0.00℃
2. 月別値から年平均を再計算 → 最大差 0.05℃（日数重みなしのため許容）
3. Sen / OLS / Spearman の3手法 → すべて正で有意
4. 気象庁公表平年値 16.9℃ vs 再計算 16.92℃ → 差 0.02℃
5. 別地点・浜松でも +0.265℃/10年（p<0.001）
6. 年次データの重複・欠落なし（86年）

詳細は `verification.md`。

## 限界と注意

- **1地点の観測データ**です。静岡県全体・日本全体を代表しません。
- 相関は因果ではありません。雷日数（+1.25日/10年）・霧日数（−0.94日/10年）の変化は、観測体制や都市化の影響を含みます。気候変動の証拠としては使いません。
- 「100年あたり」の値は分析開始年に依存します。1980年開始では +0.455℃/10年になります（`output/figures/fig4_sensitivity.png`）。
- 2026年は年途中のため除外。欠測は 1958年2・10・11月、2011年3月。
- 単調トレンドのある系列に Pettitt 検定をそのまま当てると、途中を変化点と誤判定します（生系列では1988年、p=2.8e-11）。トレンド除去後は p=0.33 で段差なし。

## 出典

> 出典：気象庁ホームページ「過去の気象データ検索」（https://www.data.jma.go.jp/stats/etrn/）を加工して作成

ライセンス: 公共データ利用規約（第1.0版）。詳細は `SOURCES.md`。
