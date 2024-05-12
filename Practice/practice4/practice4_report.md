```
University: [ITMO University](https://itmo.ru/ru/)
Faculty: [FICT](https://fict.itmo.ru)
Course: [Application containerization and orchestration](https://github.com/itmo-ict-faculty/application-containerization-and-orchestration)
Year: 2023/2024
Group: K4113c
Author: Karaulov Andrei Olegovich
Practice: practice4
Date of create: 12.05.2024
Date of finished: 12.05.2024
```
1. Были созданы тесты
```python
import requests

# Base URL for the API
BASE_URL = "http://localhost:5001"
TOKEN_ADDRESS = "E5c1ZLiMkSt46W9tvWbSR6DMQRUpkxUpkEdLRcPr9akC"


def test_get_token_info_nonexistent():
    """Test retrieval of token info for a non-existent token"""
    response = requests.get(f"{BASE_URL}/get_token_info/nonexistent_token_address")
    assert response.status_code == 404, f"Error {response.text} \n {response.json()}"
    assert "not found" in response.json()["detail"], f"Error {response.text} \n {response.json()}"


def test_add_token():
    """Test adding a new token"""
    response = requests.post(f"{BASE_URL}/add_token/{TOKEN_ADDRESS}")
    assert response.status_code == 200, f"Error {response.text} \n {response.json()}"
    assert response.json()["address"] == TOKEN_ADDRESS, f"Error {response.text} \n {response.json()}"


def test_get_token_info_existent():
    """Test retrieval of token info for an existing token using a fixture"""
    response = requests.get(f"{BASE_URL}/get_token_info/{TOKEN_ADDRESS}")
    assert response.status_code == 200
    data = response.json()
    assert isinstance(data, dict), f"Error {response.text} \n {response.json()}"
    assert (
        data["pairs"][0]["baseToken"]["address"] == TOKEN_ADDRESS
    ), f"Error {response.text} \n {response.json()}"

```
2. Был создан pipeline для запуска тестов
```yaml
name: Test Pipeline

on:
  push:
  pull_request:

jobs:
  build-and-test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Run containers with Docker Compose
        env:
          SOLANA_RPC_URL: ${{ secrets.SOLANA_RPC_URL }}
        run: |
          docker-compose -f docker-compose-test.yml up --build -d

      - name: Install PyTest
        run: |
          sudo apt-get update
          sudo apt-get install python3-pip
          pip3 install -r requirements.txt

      - name: Execute tests with PyTest
        run: |
          pytest

      - name: Collect Docker Compose logs
        if: always()
        run: |
          docker-compose -f docker-compose-test.yml logs

      - name: Upload Docker Compose logs as an artifact
        uses: actions/upload-artifact@v4
        with:
          name: docker-compose-logs
          path: docker-compose-logs.txt

      - name: Stop containers
        if: always()
        run: docker-compose -f "docker-compose.yml" down ---remove-orphans
```

3. Были добавлены doc strings для всех функций
4. Был написан README