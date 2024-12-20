## Описание программного продукта

Программа написана на языке Python. С использованием следующих библиотек: numpy, pandas, pgmpy, matplotlib, seaborn. 
* `numpy` - библиотека используется для работы с многомерными массивами, а также содержит математические функции, которые могут применяться к этим массивам.
* `pandas` - библиотека необходима для загрузки данных.
* `pgmpy` - библиотека на Python, предназначенная для работы с вероятностными графическими моделями, такими как байесовские сети и Марковские случайные поля. Ниже описываются модули, которые используются в программе.
  * `pgmpy.models.BayesianModel` используется для определения модели (структуры сети).
  * `pgmpy.models.BayesianNetwork` включает методы для работы с вероятностными распределениями.
  * `pgmpy.estimators.HillClimbSearch` является алгоритмом для поиска структуры байесовской сети. Он использует жадный алгоритм для оптимизации структуры сети, пробуя различные ребра и выбирая те, которые улучшают оценку (BIC).
  * `pgmpy.estimators.BicScore` - это функция, используемая для оценки соответствия между данными и моделью на основе байесовского информационного критерия (BIC).
* `matplotlib`, `seaborn` - библиотеки для визуализации данных.

## Формулировка задачи

**Цель работы:** предсказать вероятность сердечного приступа, исходя из соответствующих признаков.

**Предметная область:** здравоохранение.

**Ключевые слова:** обучение с учителем, байесовские сети доверия, машинное обучение в здравоохранении, математическая статистика.

**Ссылка на датасет:** https://github.com/rustam-azimov/ml-course/blob/main/data/heart_disease/heart.csv

**Описание датасета:**

* **age:** возраст пациента
* **sex:** пол пациента
* **cp:** тип боли в груди
 	- **Значение 1:** типичная стенокардия
  - **Значение 2:** атипичная стенокардия
  - **Значение 3:** неангинальная боль
  - **Значение 4:** бессимптомное течение
* **trestbps:** артериальное давление в состоянии покоя (в мм рт. ст.)
* **chol:** холестерин в мг/дл, полученный с помощью датчика ИМТ
* **fbs:** (уровень сахара в крови натощак > 120 мг/дл) (1 = true; 0 = false)
* **restecg:** результаты электрокардиографии в покое
  - **Значение 0:** нормальное
  - **Значение 1:** наличие аномалии ST-T 
  - **Значение 2:** показывает гипертрофию левого желудочка 
* **thalach:** максимальная частота сердечных сокращений
* **exang:** cтенокардия, вызванная физической нагрузкой (1 = true; 0 = false)
* **oldpeak:** Депрессия ST, вызванная физическими упражнениями, по сравнению с отдыхом
* **slope:** наклон пикового сегмента ST при нагрузке
* **ca:** количество расширенных сосудов
* **thal:** тип дефекта
  - **Значение 1:** нормальный
  - **Значение 2:** фиксированный дефект
  - **Значение 3:** обратимый дефект
* **target:** наличие сердечного приступа

## Основная часть

Перед построением байесовской сети был проведен разведочный анализ. 


- Основные статистики:

<div align="center">
  <img src="https://github.com/user-attachments/assets/ade2ac2a-4d3c-434e-af59-92eec217dcd9"/>
</div>


- Проверка на  пустые значения:
<div align="center">
  								
|    age    | sex            | cp            | trestbps   | chol       | fbs        | restecg    |thalach     |exang |	oldpeak |	slope |	ca |	thal |	target |
| --------- | -------------- | ------------- | ---------- | ---------- | ---------- | ---------- | ---------- | ---- | -------- | ----- | -- | ----- |-------- |
|0          |0               | 0             | 0          | 0          | 0          | 0          |  0         | 0    |  0       |  0    | 0  |   0   |  0      |

</div>


- **Графики плотностей распределений случайных величин:**

<div align="center">
  <img src="https://github.com/user-attachments/assets/41f01041-fdc5-44c5-8025-99c37431518d"/>
</div>

Можно сказать, что случайные величины распределены нормально, но имеют скос. Эту проблему можно решить с помощью логарифмирования случайной величины.


- **Столбчатые графики:**

<div align="center">
  <img src="https://github.com/user-attachments/assets/41b4303e-45c7-4232-b0ee-0a3dccb417b7"/>
</div>

Отсюда видно, что целевая переменная сбалансирована, чего не скажешь о факторах. 


- **Диаграммы рассеивания**

<div align="center">
  <img src="https://github.com/user-attachments/assets/9abab7c4-901e-486f-9cac-921d09a36be8"/>
</div>

- **Корреляционная матрица**

<div align="center">
  <img src="https://github.com/user-attachments/assets/73ad3f70-8ccc-443d-8cdc-f10f9c364c2d"/>
</div>

Судя по корреляционной матрице и диаграмме рассеивания факторные переменные независимы. 
После проведения разведочного анализа, с использованием критерия Фишера, были исключены ненужные признаки (fbs).

Затем была реализована байесовская сеть доверия. Для обучения структуры байесовской сети был использован метод HillClimbSearch.

```python
estimator = HillClimbSearch(data_clean)
model_structure = estimator.estimate(scoring_method=BicScore(data_clean))
```

HillClimbSearch — это один из алгоритмов поиска, используемых для построения структуры байесовских сетей. Он основывается на методе "подъема холма" (hill-climbing), что означает, что он ищет локальный максимум (или минимум), постепенно улучшая структуру сети путем добавления, удаления или изменения направлений ребер в графе.

Для оценки структуры сети будет использоваться критерий Байеса (Bayesian Information Criterion, BIC). BIC — это метод, который помогает в выборе модели, учитывая не только точность модели, но и количество параметров, что позволяет избегать переобучения.

$$BIC = k * ln(n) - 2 * ln(L)$$

где `L` - максимальное значение функции правдоподобия наблюдаемой выборки с известным числом параметров, `k` - число параметров модели, `n` - объем обучающей выборки.

Перейдём, непосредственно к модели: 

```python
model = BayesianNetwork(model_structure.edges())
model.fit(data_clean)
```

Следует пояснить, что означает класс `BayesianNetwork`. Он представляет собой структуру, где узлы сети представляют собой случайные переменные, а ребра показывают вероятностные зависимости между ними. Также стоит рассказать, что метод `edges()` возвращает список ребер (связей) в модели. В байесовской сети ребра отвечают за вероятностные зависимости между случайными переменными.

<div align="center">
  <img src="https://github.com/user-attachments/assets/61b4ddc0-f36e-45a3-ba30-e112932b6e27"/>

  <p><i>Структура байесовской сети доверия</i></p>
</div>

Продемонстрируем процесс вывода в байесовской сети, используя метод переменной элиминации (Variable Elimination).

```python
inference = VariableElimination(model)
query_result = inference.query(variables=['target'], evidence={'sex': 0,
                                                               'restecg': 1,
                                                               'slope': 2,
                                                               'ca': 2,
                                                               'thal': 3,
                                                               'oldpeak': 0.4})
print(query_result.values)
```

`Variable Elimination` - это алгоритм, который используется для выполнения вывода в байесовских сетях, позволяя вычислить распределение вероятностей для интересующих переменных, учитывая известные значения других переменных.

`inference.query` : метод, который находит распределение вероятностей для указанных переменных, учитывая предоставленные факторы.

`variables=['target']` : это переменная, для которой возвращается распределение вероятностей. 

`evidence` : здесь указываются известные значения для других переменных. Это необходимые условия, при которых будет рассчитано распределение для целевой переменной. 

Эффективность модели была  протестирована с помощью метрик accuracy, recall, precision, f1. Результат работы алгоритма приведен ниже: 

Выборка с размером 100

|	accuracy |	recall |	precision  |	f1  | roc-auc |
| -------- | ------ | ---------- |---- | ------- |
|  0.83    | 0.81   |   0.84     | 0.84|   0.83  |      

Выборка с размером 250

|	accuracy |	recall |	precision  |	f1   | roc-auc |
| -------- | ------ | ---------- |----- | ------- |
|  0.86    | 0.82   |   0.91     | 0.86 |   0.86  |    

Выборка с размером 500

|	accuracy |	recall |	precision  |	f1   | roc-auc |
| -------- | ------ | ---------- |----- | ------- |
|  0.84    | 0.81   |   0.9      | 0.86 |   0.85  |    

## Результат

Таким образом, в результате работы был проведён разведочный анализ и создана модель наивной байесовской сети. Помимо этого была проведена оценка работы созданной сети, который показал, что данная модель хорошо справляется с решением задачи классификации на анализируемом датасете. 








