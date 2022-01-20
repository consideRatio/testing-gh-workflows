# This is a GitHub workflow defining a set of jobs with a set of steps.
# ref: https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions
#
name: Test

# Insights
# 1. No dynamic needs
#    - To use a previous job's outputs, you must declare it under "needs".
#    - This leads to a failure to dynamically set the needs based on previous
#      jobs outputs that has influenced the current jobs matrix.

# Strategy idea
# - To have a few jobs that contain dynamic amounts of jobs within them, conditionally running if 
#   

on:
  push:
  workflow_dispatch:

jobs:
  # This jobs 
  initial-job:
    runs-on: ubuntu-20.04
    steps:
      id: get-matrix-include
      run: |
        echo '::set-output name=json::[
            {
                "id": "support",
                "needs": [],
                "chart": "helm-charts/support",
                "values": ["file1.yaml"]
            }
        ]'
    outputs:
      # Each separate output must be a string! Convert your more complicated
      # values into json strings first.
      matrix-include-json: '${{ steps.get-matrix-include.outputs.json }}'

  test-dynamic-matrix-values:
    runs-on: ubuntu-20.04
    strategy:
      fail-fast: false
      matrix:
        include: ${{ needs.get-matrix-include.outputs.output1 }}
    steps:
      run: |
        echo ${{ toJson(matrix) }}

  test-complicated-matrix-values:
    runs-on: ubuntu-20.04
    strategy:
      fail-fast: false
      matrix:
        include:
          - my_string: "test"
            my_number: 2
            my_bool: true
            my_json_string: '{"key": "value"}'
            my_dict:
              key: value
            my_list:
              - elem1
              - elem2
            my_needs:
              - test
    steps:
      run: |
        echo ${{ toJson(matrix) }}