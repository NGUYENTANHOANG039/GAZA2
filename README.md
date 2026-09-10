[README.md](https://github.com/user-attachments/files/32061152/README.md)

# GAZA2[HUONG-DAN.md](https://github.com/user-attachments/files/32061153/HUONG-DAN.md)[data.json](https://github.com/user-attachments/files/32061157/data.json)[index.html](https://github.com/user-attachments/files/32061162/index.html)[ict.py](https://github.com/user-attachments/files/32061169/ict.py)
[app.js](https://github.com/user-attachments/files/32061165/app.js)
[style.css](https://github.com/user-attachments/files/32061164/style.css)
[prose.py](https://github.com/user-attachments/files/32061170/prose.py)[refresh.yml](https://github.com/user-attachments/files/32061176/refresh.yml)
[data.py](https://github.com/user-attachments/files/32061174/data.py)
[__init__.py](https://github.com/user-attachments/files/32061173/__init__.py)
[scanner.py](https://github.com/user-attachments/files/32061172/scanner.py)
[publish.py](https://github.com/user-attachments/files/32061171/publish.py)
name: refresh  # 15 phút một lần: tính lại phân tích -> commit web/data.json -> Pages tự cập nhật
on:
  schedule:
    - cron: "*/15 * * * *"
  workflow_dispatch: {}
  push:
    paths:
      - "gaze/**"
      - ".github/workflows/refresh.yml"
permissions:
  contents: write
concurrency:
  group: refresh
  cancel-in-progress: true
jobs:
  build:
    runs-on: ubuntu-latest
    timeout-minutes: 6
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.11"
      - name: Tính lại + ghi web/data.json
        run: |
          python3 -m gaze.publish --out web/data.json
          python3 - <<'PY'
          import json,os
          d=json.load(open("web/data.json",encoding="utf-8"))
          n=len(d.get("symbols",{}))
          print(f"ok: {n} symbol, {len(d['symbols'][d['order'][0]]['analysis']['sections'])} mục/symbol")
          assert n>0, "không có symbol nào"
          PY
      - name: Commit (bỏ qua nếu không đổi)
        run: |
          git config user.name "gaze-refresh"
          git config user.email "bot@users.noreply.github.com"
          if [ -n "$(git status --porcelain web/data.json)" ]; then
            git add web/data.json
            git commit -m "data: $(date -u +'%Y-%m-%d %H:%M') UTC"
            git push
          else
            echo "không có gì mới"
          fi
