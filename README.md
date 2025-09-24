# California Housing — Feature Engineering & Preprocessing Pipeline

Этот репозиторий содержит ноутбук **`Cal_Has_Prisec_pipeline.ipynb`**, который строит воспроизводимый пайплайн подготовки данных по датасету *California Housing Prices* и подготавливает признаки для последующего обучения моделей регрессии.

## 📦 Данные
- Источник: [Kaggle — California Housing Prices](https://www.kaggle.com/datasets/camnugent/california-housing-prices)
- Загрузка выполняется программно через **`kagglehub`**:
  ```python
  import kagglehub
  from kagglehub import KaggleDatasetAdapter
  df = kagglehub.load_dataset(
      KaggleDatasetAdapter.PANDAS,
      "camnugent/california-housing-prices",
      "housing.csv"
  )
  ```

## 🛠️ Зависимости
Минимальный набор библиотек:
- `python>=3.9`
- `pandas`
- `numpy`
- `matplotlib`
- `scikit-learn`
- `folium`
- `kagglehub`

Быстрая установка:
```bash
pip install pandas numpy matplotlib scikit-learn folium kagglehub
```

> Примечание: для работы `kagglehub` может потребоваться настройка доступа к Kaggle (API токен).

## 🚀 Быстрый старт
1. Откройте ноутбук `Cal_Has_Prisec_pipeline.ipynb` в Jupyter/Colab.
2. Выполните ячейки сверху вниз.  
3. На выходе вы получите очищенный датафрейм с признаками, интерактивные карты Folium, нормализованные версии признаков и разбиение на train/test.

## 🔧 Структура пайплайна
1. **Импорт и загрузка данных**
   - Загрузка `housing.csv` через `kagglehub`.
2. **Первичная диагностика**
   - Подсчёт пропусков, базовые распределения числовых столбцов (`df.hist(...)`).
3. **Обработка пропусков**
   - Заполнение `total_bedrooms` медианой:
     ```python
     df["total_bedrooms"].fillna(df["total_bedrooms"].median(), inplace=True)
     ```
4. **Категориальные признаки**
   - One‑Hot Encoding для `ocean_proximity` с `drop_first=True`:
     ```python
     df = pd.get_dummies(df, columns=["ocean_proximity"], drop_first=True)
     ```
5. **Логарифмирование распределений (стабилизация хвостов)**
   - Преобразуются: `total_rooms`, `total_bedrooms`, `population`, `households` → добавляются столбцы с суффиксом `_log` через `np.log1p`.
6. **Конструирование признаков (ratio на логах)**
   - `rooms_per_household_log = total_rooms_log / households_log`
   - `bedrooms_per_room_log = total_bedrooms_log / total_rooms_log`
   - `population_per_household_log = population_log / households_log`
7. **Геопризнаки и пространственная агрегация**
   - Бинирование широты/долготы на равные интервалы (`pd.cut`) и агрегация медианы `median_house_value` в ячейках сетки.
   - Визуализация сетки **Folium** (`folium.Rectangle`) с градиентом по медианной цене; отдельная карта выбросов по `population_per_household_log`.
8. **Целевая переменная**
   - Добавление `median_house_value_log = log1p(median_house_value)` и сравнение распределений исходной и лог‑цели.
9. **Масштабирование признаков**
   - `StandardScaler` → `X_sscaled` (все признаки `X` кроме целевой).
   - `MinMaxScaler(0..1)` → `X_mmscaled` только для числовых столбцов.
   - Сравнение гистограмм «оригинал vs StandardScaler vs MinMaxScaler`».
10. **Разбиение на выборки**
    ```python
    X = df.drop(columns=["median_house_value_log"])
    y = df["median_house_value_log"]
    X_train, X_test, y_train, y_test = train_test_split(
        X, y, test_size=0.2, random_state=42, shuffle=True
    )
    ```

## 🧱 Ключевые признаки на выходе
- Базовые числовые признаки из датасета (кроме удалённых оригиналов после лог‑преобразований).
- One‑Hot столбцы по `ocean_proximity` (с `drop_first`).
- Лог‑признаки: `*_log` для: `total_rooms`, `total_bedrooms`, `population`, `households`.
- Сводные ratio‑признаки на логах:
  - `rooms_per_household_log`
  - `bedrooms_per_room_log`
  - `population_per_household_log`
- Цель: `median_house_value_log`.

## 🌍 Визуализации
- Гистограммы распределений для диагностики «до/после».
- Две интерактивные карты Folium:
  1) медианные цены по пространственной сетке,
  2) отображение «выбросных» ячеек по `population_per_household_log`.

## ⚙️ Параметры, которые можно настраивать
- `test_size=0.2` и `random_state=42` в `train_test_split`.
- Количество бинов по широте/долготе (в ноутбуке — равномерная сетка, по умолчанию 100×100).
- Порог для отбора «выбросов» по `population_per_household_log` (в ноутбуке — эквивалент `> log1p(100)`).

## 📄 Артефакты/переменные
- `df` — итоговый датафрейм с признаками и целевой.
- `X_sscaled`, `X_mmscaled` — масштабированные матрицы признаков (DataFrame).
- `X_train, X_test, y_train, y_test` — разбиение для обучения/валидации.
- Интерактивные объекты **Folium** `m` (карта сетки) и карта «выбросов».

## 🧪 Что дальше?
- Добавить обучение базовой модели (например, `LinearRegression`) на `X_train/y_train`, метрики `RMSE` и `R²` на `X_test/y_test`.
- Сравнить качество на сырой, StandardScaler и MinMaxScaler версиях.
- Экспорт модели (`joblib`/`pickle`) и полноценный инференс‑скрипт.

---

**Лицензия:** MIT (или укажите вашу).  
**Автор:** (вставьте своё имя/ник).

