# Equity Structure Analysis Platform

An NLP pipeline that reads the **annual reports of Chinese A-share listed companies** (Shanghai Stock Exchange) and automatically draws **interactive ownership-structure diagrams**: each company's major shareholders and subsidiaries, labelled by domestic/overseas status and business sector.

Built for the Citi Financial Innovation Application Competition (Citi China Markets Challenge), Jan – May 2024. Qimu Luo was team leader and business analyst.

🎬 Demo video: [`Demo-Equity-Structure.mp4`](Demo-Equity-Structure.mp4)

## Pipeline

```
annual report (.docx)
   │  python-docx: extract every table and the text around it
   ▼
BERT table classifier ─── finds the shareholder table (股东表) and the subsidiary table (子公司表)
   │                       among all tables in the report
   ▼
table cleaning ────────── merges tables split across pages, drops repeated headers,
   │                       keeps name / shareholding ratio / business columns
   ▼
entity labelling ──────── • shareholders: domestic vs. overseas (境内 / 境外)
   │                       • subsidiaries: region of operation, domestic vs. overseas (character-level PyTorch classifier)
   │                       • subsidiaries: business sector, 18 CSRC industry classes + "undetermined" (BERT classifier)
   │                       • company-name segmentation with LTP
   ▼
Graphviz diagram ──────── shareholders → company → subsidiaries, exported as SVG,
                           with JavaScript added so nodes can be clicked to expand or collapse groups
```

The package also contains the **training code** for the classifiers (`create_train_and_test_set`, `train_for_finding_tables`, `train_for_identify_Overseas_Entities`). It also includes a helper, `downloadreport_year`, that downloads annual reports from the SSE disclosure site.

## Repository structure

```
Demo-Equity-Structure.mp4
code/
├── Demo_code.py                         # entry point: runs the full pipeline on a folder of .docx reports
├── Docx2OSD/                            # "docx to ownership-structure diagram" package
│   ├── __init__.py
│   └── Judge_and_draw_src.py            # extraction, classification, cleaning, drawing, training
├── models_find_tables/vocab.txt         # BERT vocabulary (table classifier)
├── model_business_classification/vocab.txt
├── models_Judge_regions_of_composed_comanys/
│   ├── judge_Regions_of_Operation_of_composedcomp.pth   # region-of-operation classifier weights
│   └── extended_dict_of_judge_Regions_model.json        # its character vocabulary
├── docxs_used_to_test/                  # sample 2022–2023 annual reports (.docx)
├── requirements_cpu.txt
└── requirements_withcuda.txt
```

## Usage

```bash
cd code
pip install -r requirements_cpu.txt     # or requirements_withcuda.txt
# Graphviz must also be installed on the system (e.g. `brew install graphviz` / `apt install graphviz`)
python Demo_code.py
```

`Demo_code.py` runs `Docx2OSD.Judge_and_draw('docxs_used_to_test')`, which:

1. classifies the tables in each report and saves them to `xlsx被模型标记完成的/`;
2. merges the shareholder and subsidiary tables into `xlsx包括合成好的三张表/`;
3. writes one interactive diagram per company to `svg_graphs/`. Open these in a browser.

**Model weights:** the two fine-tuned BERT checkpoints that the code loads, `models_find_tables/model_find_tables.pth` and `model_business_classification/model_business_classification4.pth`, are not included in this repository. To run the full pipeline, either put the checkpoints at those paths or retrain them with the training functions in `Docx2OSD`. On first run, `transformers` and `ltp` also download the `bert-base-uncased` config and the LTP model.

## Tech stack

Python · PyTorch · Hugging Face Transformers (BERT) · LTP · python-docx · pandas / openpyxl · Graphviz · SVG + JavaScript
