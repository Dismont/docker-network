FROM python
COPY . /app
WORKDIR /app
RUN pip install -r requirements.txt
ENTRYPOINT [ "python3" ]
CMD [ "app.py" ]