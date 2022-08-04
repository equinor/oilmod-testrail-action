# Junit -> Testrail action

Usage:
```yaml
name: Run tests and upload

on:
  pull_request:
    branches: [ main ]

jobs:
  test:
    defaults:
      run:
        shell: bash
        working-directory: client/
    name: Run unit tests
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v2
        with:
          node-version: '14'
      - name: Install
        run: npm install
      - name: Run tests
        run: npm run tests
      - name: Create junit report # Optional step - this step will annotate the workflow summary with test results
        uses: mikepenz/action-junit-report@v3
        if: always() # always run even if the previous step fails - otherwise it wont run for test-failures
        with:
          report_paths: './client/dist/jest/reports/*.xml' # Include *.xml here
          check_name: 'Test name in summary'
      - name: Upload to testrail
        if: always() # always run even if the previous step fails - otherwise it wont run for test-failures
        uses: equinor/oilmod-testrail-action@v1
        continue-on-error: true # Avoid failing the job if upload fails
        with:
          testrail-user: <your testrail email>
          testrail-password: ${{ secrets.TESTRAIL_PASSWORD }}
          path-to-files: "./client/dist/jest/reports"
          testrail-project: "Oiltrade Enterprise Testautomation"
          # Title of test run. ${{ github.ref_name }} will append the ref (PR#, branch name etc)
          testrail-run-title: "<Project name> <Test suite name> - ${{ github.ref_name }}"
          # ID of test suite in testrail that you want to group these tests under
          testrail-suite-id: 000 
          # run-reference is added as extra information to the test run in testrail
          run-reference: https://github.com/${{ github.repository }}/actions/runs/${{ github.run_id }} - ${{ github.ref_name }}
```