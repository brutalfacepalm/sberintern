# sberintern

#### Эксперименты с SARIMA и библиотекой SKtime лежат в блакнотике Sarim_SKtime.ipynb ####
#### Эксперименты с нейронными сетями лежат на [Google Collab](https://colab.research.google.com/drive/1UmVPYV3gCJUPoO-uTlLq3FpMClTzlblx#scrollTo=UHs19f9dd3Zt&uniqifier=1) ####

#### Итогова модель: ####
Генерируется различные фичи, по типу MA, EMA и тд, всего их 75 штук. \
Для генерации фичей нужен запас данных в 91 день. \
Для работы модели нужны 31 день, в качестве исторических данных и 30 дней для прогноза на каждый день.  \
Для простоты принимается что в каждом месяце 30 дней.  \

BiLSTM с 2 слоями и механизмом attention, размерность скрытого слоя 512.  \
Дополнительно 3 слоя Conv1D с размером ядра 9 и padding 4, с BatchNorm и GlobalMaxPooling в конце.  \
Поверх этого 12 линейных голов с активацией SiLU и Dropout(0.2) для предсказания на 1, 2 и тд до 12 месяцев вперед.  \
Число параметров для тренировки: 70_337_145 \
Обучение в 2 этапа - 300 и 180 эпох \
Loss - MSE(L1Loss) \
Оптимизация разбита по слоям, один для BiLSTM+Conv1D и 12 для линейных голов. \
Оптимизаторы - AdamW, LR = 0.001(0.0008), weight_decay = 0.1(0.3) \
Отдельно для оптимизатора на BiLSTM+Conv1D Scheduler на основе CosineAnnealingWarmRestarts: в течении 20 эпох понижение LR в 100 раз, каждые 60 эпох понижение границ для CosineAnnealingWarmRestarts в 2 раза. \
Оптимизаторы для голов без шедулера. \

#### Веса можно скачать тут: [Обученная модель](https://drive.google.com/file/d/164Rkvu7Ej3x5C9efL6Dp2ocsNDnaYGsK/view?usp=sharing) и [StandartScaler](https://drive.google.com/file/d/1lEVf2yECUB-022q-6O_KHxd_6bsFGO8I/view?usp=sharing) ####

#### Так как модель предсказывает по одному значению на 1, 2 и тд до 12 месяцев вперед, то за один прогон прогнозирования на 30 днях исторических данных получается 360 дней прогноза. Но на больших периодах ошибка большая, поэтому прогнозирование делается 12 раз(первый раз на исторических данных, последующие разы на уже спронозированных) усредняя результаты по маске, зависящей от номера месяца прогнозирования. ####
Например для 3 месяцев маска по возрастанию выглядит как \[0.58997485, 0.31169601, 0.09832914\]
#### Дополнительно после предсказания и маскирования делается экспоненциальное сглаживание с окном 3 ####
#### По финальному прогнозу берется min и умножается на коэффициент, по умолчанию 0.9 ####