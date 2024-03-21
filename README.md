ctr_project
==============================

Работа над данным проектом происходила в рамках прохождения курса по MLOps. В мире онлайн-рекламы кликабельность (Click-Through Rate или CTR) является очень важной метрикой для оценки эффективности рекламы. 
В связи с этим, системы предсказания кликов имеют большое значение и широко используются для спонсорского поиска 
и ставок в режиме реального времени (real time bidding).

В данном проекте мы построим production-ready пайплайн по предсказанию кликов пользователя для мобильной Web рекламы.
За основу возьмем данные соревнования Kaggle [Avazu CTR Prediction](https://www.kaggle.com/competitions/avazu-ctr-prediction/overview/description).


## Описание пайплайна
Пайплайн с моделью состоит из трех основных элементов
- `make_dataset`: чтения данных
- `features/build_transformers`: обработки признаков, в которую входят
  - `DeviceCountTransformer`, `UserCountTransformer`: трансформы для расчета количества 
рекламных объявлений на пользователя или девайс
  - `CtrTransformer`: трансформы, с помощью которых кодируем категориальные переменные средним CTR
- `model_fit_predict`: обучаем классическую модель `Catboost` `а на предсказание вероятности клика для данной сессии пользователя. 

Поскольку целью данного курса являются не сами эксперименты или файнтюнинг модели, а построение пайплайна
то будем исходить из предположения, что это некоторая готовая версия модели, и нас просят катить ее в прод.
Поэтому мы сосредоточим усилия на воспроизводимости, поддерживаемости, развертке и мониторинге.


### Установка 
```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

### Обучение модели
```bash
python ctr_project/train_pipeline.py --configs configs/train_config.yaml
```

### Юнит тестирование:
```bash
pytest
```

### Результаты
Наш пайплайн будет достаточно прямолинейным и будет содержать элементы описанные ранее.
Кастомные обработки фичей обернуты в формат `sklearn transformer` для единооборазия.

### Основные команды для запуска сервиса с удаленной машины
```bash
# resrict rights for .pem key pair VM access file
chmod 400 your_key_pair.pem

# connect to VM by SSH
ssh -i your_key_pair.pem ubuntu@your_external_ip

# pull docker image from docker hub
docker pull evgeniimunin/ctr_online_inference:v1

# run fastapi app in docker container
docker run -p 8000:8000 evgeniimunin/ctr_online_inference:v1
```

### Результаты
После написания REST сервиса на FastAPI мы можем его протестировать, отправив запросы на инференс.
В качестве ответа мы получим код 200, означающий успешный ответ с сервера, а тело ответа будет содержать
`device_ip` и предсказанную вероятность клика для данной сессии `click_proba`.
```
response.status_code: 200
response.json(): [{'device_ip': '7061f023', 'click_proba': 0.2136}]
```

## Организация проекта
```
    ├── LICENSE
    ├── Makefile           <- Makefile with commands like `make data` or `make train`
    ├── README.md          <- The top-level README for developers using this project.
    ├── data
    │   ├── external       <- Data from third party sources.
    │   ├── interim        <- Intermediate data that has been transformed.
    │   ├── processed      <- The final, canonical data sets for modeling.
    │   └── raw            <- The original, immutable data dump.
    │
    ├── docs               <- A default Sphinx project; see sphinx-doc.org for details
    │
    ├── models             <- Trained and serialized models, model predictions, or model summaries
    │
    ├── notebooks          <- Jupyter notebooks. Naming convention is a number (for ordering),
    │                         the creator's initials, and a short `-` delimited description, e.g.
    │                         `1.0-jqp-initial-data-exploration`.
    │
    ├── references         <- Data dictionaries, manuals, and all other explanatory materials.
    │
    ├── reports            <- Generated analysis as HTML, PDF, LaTeX, etc.
    │   └── figures        <- Generated graphics and figures to be used in reporting
    │
    ├── requirements.txt   <- The requirements file for reproducing the analysis environment, e.g.
    │                         generated with `pip freeze > requirements.txt`
    │
    ├── setup.py           <- makes project pip installable (pip install -e .) so src can be imported
    ├── src                <- Source code for use in this project.
    │   ├── __init__.py    <- Makes src a Python module
    │   │
    │   ├── data           <- Scripts to download or generate data
    │   │   └── make_dataset.py
    │   │
    │   ├── features       <- Scripts to turn raw data into features for modeling
    │   │   └── build_features.py
    │   │
    │   ├── models         <- Scripts to train models and then use trained models to make
    │   │   │                 predictions
    │   │   ├── predict_model.py
    │   │   └── train_model.py
    │   │
    │   └── visualization  <- Scripts to create exploratory and results oriented visualizations
    │       └── visualize.py
    │
    └── tox.ini            <- tox file with settings for running tox; see tox.readthedocs.io
```

<p><small>Project based on the <a target="_blank" href="https://drivendata.github.io/cookiecutter-data-science/">cookiecutter data science project template</a>. #cookiecutterdatascience</small></p>
