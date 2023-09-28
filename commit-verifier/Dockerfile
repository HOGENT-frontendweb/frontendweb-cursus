FROM python:3-alpine

WORKDIR /app

COPY requirements.txt ./requirements.txt
RUN pip install -r ./requirements.txt

COPY commit-verifier.py ./commit-verifier.py

ENTRYPOINT ["python", "./commit-verifier.py"]
