name: Update profile art

on:
  schedule:
    - cron: "17 3 * * *"   # daily, off-the-hour to avoid GH Actions rush
  workflow_dispatch: {}     # allows manual "Run workflow" trigger

permissions:
  contents: write

jobs:
  refresh:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.12"

      - name: Install scraper deps
        run: pip install -r scripts/requirements.txt

      - name: Fetch contributions
        env:
          GH_PROFILE_USER: ${{ github.repository_owner }}
        run: python scripts/fetch_contributions.py

      - name: Render heatmap
        run: python scripts/render_heatmap_svg.py

      - name: Commit if changed
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git add data/contributions.json contrib-heatmap.svg
          if git diff --cached --quiet; then
            echo "No changes to commit."
          else
            git commit -m "chore: refresh contribution heatmap"
            git push
          fi
