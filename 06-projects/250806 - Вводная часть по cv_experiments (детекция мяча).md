
## Вводная часть 
### Полезные ссылки 
1. [cv_experiments](https://github.com/uZlnckT2AmRfqD0UcJcp/cv_experiments) - репозиторий, в котором представлен код пайплайна для обучения под задачу выделения объектов
2. [cv_configs](https://github.com/uZlnckT2AmRfqD0UcJcp/cv_configs) -  репозиторий, в котором представлены конфигурационные файлы
3. [clearml](https://app.soccer-clearml.lobachevsky-sv.ru) 

### Настройка ClearML
После выполнения [первоначальной настройки ClearML](https://clear.ml/docs/latest/docs/clearml_sdk/clearml_sdk_setup/#local-python) необходимо обновить значения поля `default_cache_manager_size` до некого большого числа (например, `10000000`) - это необходимо для того, чтобы кэш не перетирался по мере подгрузки новых данных.

## Cуществующие наработки
### Наборы данных 
Все данные хранятся в yolo-подобной формате. 
- [Набор открытых данных в clearml](https://app.soccer-clearml.lobachevsky-sv.ru/projects/672e35e65d744b5fad7a4ddb5e6402c9/tasks/5573b38a7da64617b4c370677642fa18/execution?columns=selected&columns=name&columns=tags&columns=status&columns=project.name&columns=users&columns=started&columns=last_update&columns=type&columns=last_iteration&columns=parent.name&columns=hyperparams.dataset_info.names&columns=hyperparams.dataset_info.number_samples&order=-last_update&filter=tags:class%253Aball&deep=true)
	По ссылке представлен список открытых данных которые удалось найти, при этом не факт, что он полный.
- Набор размеченных данных
	Разметка производилась на базе кадров из записей матчей ЦСКА с технической камеры. Кадры из трансляций брались случайным образом, но при этом гарантировалось, что для одной трансляции кадр с фиксированным порядковым номером будет не более чем в одном наборе данных. 
	При этом существует два варианта представления размеченных данных:
	1. Наборы разбитые по тому, из какого видео были взяты кадры + последовательный номер набора данных.
		[ссылка на список датасетов](https://app.soccer-clearml.lobachevsky-sv.ru/projects/09698afb08b341af849f1fb9d99f5bcd/tasks/6519186e98d3441a85a73f0866b2b3c7/execution?columns=selected&columns=name&columns=tags&columns=status&columns=project.name&columns=users&columns=started&columns=last_update&columns=type&columns=last_iteration&columns=parent.name&columns=hyperparams.dataset_info.number_samples&order=-started&filter=tags:class%253Aball&deep=true)
	2. Наборы данных сгруппированные по итерации разметки.
		[ссылка на список датасетов](https://app.soccer-clearml.lobachevsky-sv.ru/projects/4ebeff39e53b447285bdfd6191f0eef8/tasks/cdf6bda459124f66ae00b83703c11c84/execution?columns=selected&columns=name&columns=tags&columns=status&columns=project.name&columns=users&columns=started&columns=last_update&columns=hyperparams.dataset_info.number_samples&columns=hyperparams.dataset_info.names&columns=type&columns=last_iteration&columns=parent.name&order=-last_update&filter=tags:class%253Aball&deep=true)

### Пайплайн по выделению мяча
Пайплайн для обучения хранится в репозитории [`cv_experiments`](https://github.com/uZlnckT2AmRfqD0UcJcp/cv_experiments), а кодовую базу можно найти по пути [`experiments/train/object_detection/main_pipeline/`](https://github.com/uZlnckT2AmRfqD0UcJcp/cv_experiments/tree/main/experiments/train/object_detection/main_pipeline).
#### Описание пайплайна
Базовый пайплайн состоит из 
1. `get_config` - Загрузка конфигурационного файла
2. `process_config` - Валидация и структурирование конфигурации
3. `prepare_data` - Подготовка и загрузка данных для обучения/тестирования
4. `add_tags_to_pipeline` - Добавление тегов к задаче пайплайна в ClearML
5. `convert_data` - Конвертация данных в формат, требуемый моделью
6. `train_model` - Обучение модели
7. `test_model` - Тестирование обученной или предобученной модели

Для возможности подменить модель в пайплайне были реализованы:
- специальная обертка для моделей
  `main_pipeline/utils/model_wrapper/base_model.py
- специальная обертка для подготовки данных 
  `main_pipeline/utils/dataset_converter/base_converter.py`

#### Запуск

Для запуска необходимо выбрать одну из 4-ех возможных конфигураций:
- `default` 
- `local_debug`
- `local_main`
- `remote_debug`

Два главных параметра:
- `local` / `remote` - режим запуска
	- `local` - локальный запуск
	- `remote` - запуск на удаленном агенте
- `main` / `debug` - то, куда сохранится информация о запуске, на примере пайплана выделения объектов (`<project>/<mode>/pipeline/object_detection/`) этот параметр влияет на значение `<mode>`
	- `main` - используется для продуктовых-запусков, когда пайплайн выполняется с корректными параметрами, на реальных данных.
	- `debug` - используется для тестирования, экспериментов, поиска багов локальной разработки.

Выбор конфигурации осуществляется с помощью инициализации переменной окружения `ENV_FOR_DYNACONF`. Подробную информацию о каждой конфигурации можно найти в файле `main_pipeline/settings.toml`

Пример команды для запуска:
```bash
# Запуск из корня директории
export ENV_FOR_DYNACONF=local_debug
PYTHONPATH="." python3 experiments/train/object_detection/main_pipeline/clearm_pipeline.py \
        --config_name aicska/object_detection/ultralytics/player/250419_1141_yolov11_train.yaml 
        --model_type ultralytics \
        --project aicska
```

где:
- `config_name` - относительный путь до конфигурационного файла с настройками пайплайна
- `model_type` - название модели, список доступных моделей определен на уровне `main_pipeline/utils/model_wrapper/__init__.py`
- `project` - то, куда сохранится информация о запуске, на примере пайплана выделения объектов (`<project>/<mode>/pipeline/object_detection/`) этот параметр влияет на значение `<project>`

#### Конфигурационные файлы
С помощью файла конфигурации можно задавать параметры пайплайна, причем есть фиксированные параметры, которые необходимы для работы пайплайна (`tags`, `model_type`, `train_data`, ...), а есть не фиксированные параметры, которые необходимы для конфигурации модели (`train_config`).

Список обязательных полей обозначен на уровне файла `main_pipeline/stages/check_config.py`

```
tags:
- ball
model_type: yolov5

train_data:
	datasets:
	- b80d91db5c1d47d9bd74ccebe154e927 # class:ball-markup:our-001
	- 968c638b4a4c471ab5f2f0f29e61e6d4 # class:ball-markup:tagme-001-02
	- 9a3d08334e124699b15b3839ed5857f9 # class:ball-markup:tagme-002
	- 7668f7963aec46b2980db513e68ae206 # class:ball-markup:tagme-003

test_data:
	datasets:
	- b80d91db5c1d47d9bd74ccebe154e927 # class:ball-markup:our-001
	- 968c638b4a4c471ab5f2f0f29e61e6d4 # class:ball-markup:tagme-001-02
	- 9a3d08334e124699b15b3839ed5857f9 # class:ball-markup:tagme-002
	- 7668f7963aec46b2980db513e68ae206 # class:ball-markup:tagme-003
	- e66af6c36c074eb794c0dee6fa6ca1e4 # soccernet_ball_001

train_config:
	weights: yolov5m6.pt
	cfg: ""
	hyp:
	lr0: 0.01
	lrf: 0.01

test_batch_size: 12
```

Отдельно стоит отметить, что конфигурационные файлы для обучения хранятся в отдельном репозитории, это необходимо для корректной работы ClearML-агента. Такой подход позволяет менять конфигурационный файл не изменяя основной репозиторий с пайплайном обучения. При каждом запуске ClearML-агент подтягивает актуальную версию sub-репозитория `configs` 

### Обученные модели
Обученные модели можно найти в ClearML, ниже представлены две ссылки на модели:
1. [Лидерборд - ultralytics](https://app.soccer-clearml.lobachevsky-sv.ru/pipelines/54c1c7da8dc249c49bed01adc4f3a63d/tasks?columns=selected&columns=name&columns=comment&columns=tags&columns=status&columns=started&columns=last_update&columns=project.name&columns=m.098f6bcd4621d373cade4e832627b4f6.5d37e8067872af42d2ef1fddd22ccd69.value.test.class%253Aball-markup%253Aour-001-ver%253A1%252e0%252e0%252FmAP%25400%252e5&columns=m.098f6bcd4621d373cade4e832627b4f6.f445d1ab46759daa23a428ed5b5d0e77.value.test.class%253Aball-markup%253Aour-001-ver%253A1%252e0%252e0%252FmAP%25400%252e5%253A0%252e95&columns=m.098f6bcd4621d373cade4e832627b4f6.e5a6ccd12dcdc644676be2227358f1c7.value.test.class%253Aball-markup%253Atagme-001-02-ver%253A1%252e0%252e2%252FmAP%25400%252e5&columns=m.098f6bcd4621d373cade4e832627b4f6.b1c980ed95565f1e6328dc77b131342b.value.test.class%253Aball-markup%253Atagme-001-02-ver%253A1%252e0%252e2%252FmAP%25400%252e5%253A0%252e95&columns=m.098f6bcd4621d373cade4e832627b4f6.e01aefe7b0e34437bf8c7c836e6431ff.value.test.class%253Aball-markup%253Atagme-002-ver%253A1%252e0%252e0%252FmAP%25400%252e5&columns=m.098f6bcd4621d373cade4e832627b4f6.c2f40ba97703704d05c72cde143e969f.value.test.class%253Aball-markup%253Atagme-003-ver%253A1%252e0%252e0%252FmAP%25400%252e5&columns=m.098f6bcd4621d373cade4e832627b4f6.eef14e0c6b5d90c1d6f8463f34e13cb3.value.test.class%253Aball-markup%253Atagme-002-ver%253A1%252e0%252e0%252FmAP%25400%252e5%253A0%252e95&columns=m.098f6bcd4621d373cade4e832627b4f6.f3769c28cd8d697c322c74808001228a.value.test.class%253Aball-markup%253Atagme-003-ver%253A1%252e0%252e0%252FmAP%25400%252e5%253A0%252e95&columns=m.098f6bcd4621d373cade4e832627b4f6.4e571566f33117c033becea91b94dd37.value.test.soccernet_ball_001-ver%253A1%252e0%252e2%252FmAP%25400%252e5&columns=m.098f6bcd4621d373cade4e832627b4f6.330334d2c9dad643c38b3c0c1d898bca.value.test.soccernet_ball_001-ver%253A1%252e0%252e2%252FmAP%25400%252e5%253A0%252e95&columns=m.3a6d0284e743dc4a9b86f97d6dd1a3bf.4e571566f33117c033becea91b94dd37.value.val.soccernet_ball_001-ver%253A1%252e0%252e2%252FmAP%25400%252e5&columns=m.3a6d0284e743dc4a9b86f97d6dd1a3bf.330334d2c9dad643c38b3c0c1d898bca.value.val.soccernet_ball_001-ver%253A1%252e0%252e2%252FmAP%25400%252e5%253A0%252e95&order=-started&filter=tags:ball)
2. [Лидерборд - yolov5](https://app.soccer-clearml.lobachevsky-sv.ru/pipelines/e3dba62ed5234edcb8dc62d6cfdbcf8b/tasks?columns=selected&columns=name&columns=tags&columns=status&columns=project.name&columns=started&columns=hyperparams.properties.version&columns=m.098f6bcd4621d373cade4e832627b4f6.5d37e8067872af42d2ef1fddd22ccd69.value.test.class%253Aball-markup%253Aour-001-ver%253A1%252e0%252e0%252FmAP%25400%252e5&columns=m.098f6bcd4621d373cade4e832627b4f6.f445d1ab46759daa23a428ed5b5d0e77.value.test.class%253Aball-markup%253Aour-001-ver%253A1%252e0%252e0%252FmAP%25400%252e5%253A0%252e95&columns=m.098f6bcd4621d373cade4e832627b4f6.e5a6ccd12dcdc644676be2227358f1c7.value.test.class%253Aball-markup%253Atagme-001-02-ver%253A1%252e0%252e2%252FmAP%25400%252e5&columns=m.098f6bcd4621d373cade4e832627b4f6.b1c980ed95565f1e6328dc77b131342b.value.test.class%253Aball-markup%253Atagme-001-02-ver%253A1%252e0%252e2%252FmAP%25400%252e5%253A0%252e95&columns=m.098f6bcd4621d373cade4e832627b4f6.e01aefe7b0e34437bf8c7c836e6431ff.value.test.class%253Aball-markup%253Atagme-002-ver%253A1%252e0%252e0%252FmAP%25400%252e5&columns=m.098f6bcd4621d373cade4e832627b4f6.eef14e0c6b5d90c1d6f8463f34e13cb3.value.test.class%253Aball-markup%253Atagme-002-ver%253A1%252e0%252e0%252FmAP%25400%252e5%253A0%252e95&columns=m.098f6bcd4621d373cade4e832627b4f6.c2f40ba97703704d05c72cde143e969f.value.test.class%253Aball-markup%253Atagme-003-ver%253A1%252e0%252e0%252FmAP%25400%252e5&columns=m.098f6bcd4621d373cade4e832627b4f6.f3769c28cd8d697c322c74808001228a.value.test.class%253Aball-markup%253Atagme-003-ver%253A1%252e0%252e0%252FmAP%25400%252e5%253A0%252e95&columns=m.098f6bcd4621d373cade4e832627b4f6.330334d2c9dad643c38b3c0c1d898bca.value.test.soccernet_ball_001-ver%253A1%252e0%252e2%252FmAP%25400%252e5%253A0%252e95&columns=m.290612199861c31d1036b185b4e69b75.5170afde62fc9044f329427e35eee28c.value.Summary.test%252Fclass%253Aball-markup%253Aour-001-ver%253A1%252e0%252e0%252FR&columns=m.290612199861c31d1036b185b4e69b75.da63b3e46d3b174559cf97d56f8de66d.value.Summary.test%252Fclass%253Aball-markup%253Aour-001-ver%253A1%252e0%252e0%252FmAP50&columns=m.290612199861c31d1036b185b4e69b75.8c13827fec30e971b0c667a4bff80ce3.value.Summary.test%252Fclass%253Aball-markup%253Aour-001-ver%253A1%252e0%252e0%252FmAP50-95&columns=m.290612199861c31d1036b185b4e69b75.640e9f8725b9ce3cae6615e75435207b.value.Summary.test%252Fclass%253Aball-markup%253Aour-001-ver%253A1%252e0%252e0%252FmAP%25400%252e5&columns=m.290612199861c31d1036b185b4e69b75.9a328262f61aa56a5e5f42ba4101aea8.value.Summary.test%252Fclass%253Aball-markup%253Aour-001-ver%253A1%252e0%252e0%252FmAP%25400%252e5%253A0%252e95&columns=m.290612199861c31d1036b185b4e69b75.80c703d4fe711dc568be1615b849671f.value.Summary.test%252Fclass%253Aball-markup%253Atagme-001-02-ver%253A1%252e0%252e0%252FP&columns=m.290612199861c31d1036b185b4e69b75.1378a46e7589a05a5a1c01a48a26f617.value.Summary.test%252Fclass%253Aball-markup%253Atagme-001-02-ver%253A1%252e0%252e0%252FR&columns=m.290612199861c31d1036b185b4e69b75.9ab879e41f9ee73fc26a7175d226d7c1.value.Summary.test%252Fclass%253Aball-markup%253Atagme-001-02-ver%253A1%252e0%252e0%252FmAP50&columns=m.290612199861c31d1036b185b4e69b75.82735d0d218557883ecb728080189864.value.Summary.test%252Fclass%253Aball-markup%253Atagme-001-02-ver%253A1%252e0%252e0%252FmAP50-95&columns=m.290612199861c31d1036b185b4e69b75.a64e4f375eae90d5491a143fe507a644.value.Summary.test%252Fclass%253Aball-markup%253Atagme-001-02-ver%253A1%252e0%252e0%252FmAP%25400%252e5&columns=m.290612199861c31d1036b185b4e69b75.524a39bcdd919290ef3cc18267e1b13e.value.Summary.test%252Fclass%253Aball-markup%253Atagme-001-02-ver%253A1%252e0%252e0%252FmAP%25400%252e5%253A0%252e95&columns=m.290612199861c31d1036b185b4e69b75.3b51ea94a084065c0a65ebab64f2252f.value.Summary.test%252Fclass%253Aball-markup%253Aour-001-ver%253A1%252e0%252e0%252FP&columns=m.290612199861c31d1036b185b4e69b75.f84224faa239da1c0e78b74c8af30d4e.value.Summary.metrics%252Frecall&columns=m.290612199861c31d1036b185b4e69b75.4103eccf64b989c4182efcb811a2dbdb.value.Summary.metrics%252Fprecision&columns=m.290612199861c31d1036b185b4e69b75.9a2e39977295ef10a04205799652f119.value.Summary.test%252Fsoccernet_ball_001-ver%253A1%252e0%252e2%252FmAP50-95&columns=m.290612199861c31d1036b185b4e69b75.d9b0b3da6f447d798545d8d6187bb87e.value.Summary.test%252Fclass%253Aball-markup%253Atagme-003-ver%253A1%252e0%252e0%252FmAP50-95&columns=m.290612199861c31d1036b185b4e69b75.e6e53e10571738452d06baa28865fa75.value.Summary.test%252Fclass%253Aball-markup%253Atagme-003-ver%253A1%252e0%252e0%252FmAP50&columns=m.290612199861c31d1036b185b4e69b75.5f9406b86299c166b3136ac03a81c68e.value.Summary.test%252Fclass%253Aball-markup%253Atagme-003-ver%253A1%252e0%252e0%252FR&columns=m.290612199861c31d1036b185b4e69b75.f85ce2153d5994ae06a4694eb57cf243.value.Summary.test%252Fclass%253Aball-markup%253Atagme-002-ver%253A1%252e0%252e0%252FmAP%25400%252e5%253A0%252e95&columns=m.290612199861c31d1036b185b4e69b75.a5fc925102cab0a5e34f39bc2439042e.value.Summary.test%252Fclass%253Aball-markup%253Atagme-002-ver%253A1%252e0%252e0%252FmAP%25400%252e5&columns=m.290612199861c31d1036b185b4e69b75.5d7fe165ff9f562a0e6bfa701546d678.value.Summary.test%252Fclass%253Aball-markup%253Atagme-002-ver%253A1%252e0%252e0%252FmAP50-95&columns=m.290612199861c31d1036b185b4e69b75.54eb9490c6406627c2fffd837a1ed2fb.value.Summary.test%252Fclass%253Aball-markup%253Atagme-001-ver%253A1%252e0%252e0%252FmAP50-95&columns=m.290612199861c31d1036b185b4e69b75.f19b47e79c7a4651acc04d58dffdee18.value.Summary.test%252Fclass%253Aball-markup%253Atagme-001-ver%253A1%252e0%252e0%252FmAP50&columns=m.290612199861c31d1036b185b4e69b75.aef5a2fb6d6362ca8efca61826714f4e.value.Summary.test%252Fclass%253Aball-markup%253Atagme-001-ver%253A1%252e0%252e0%252FR&columns=m.290612199861c31d1036b185b4e69b75.3bc2ab73360bfeabb5b1025949b79ab2.value.Summary.test%252Fclass%253Aball-markup%253Atagme-001-ver%253A1%252e0%252e0%252FP&columns=m.290612199861c31d1036b185b4e69b75.889f87a0c913bc5ed3d6abec2a757143.value.Summary.test%252Fsoccernet_ball_001-ver%253A1%252e0%252e2%252FmAP%25400%252e5&columns=m.098f6bcd4621d373cade4e832627b4f6.4e571566f33117c033becea91b94dd37.value.test.soccernet_ball_001-ver%253A1%252e0%252e2%252FmAP%25400%252e5&order=-started&filter=tags:ball)