

**Проект**:

Запрос сформирован тем, что просмотр фильмов на оригинальном языке - это популярный и действенный метод упражнений по изучению иностранных языков. Важно выбрать фильм, который подходит студенту по уровню сложности, т.е. студент понимал 50-70 % диалогов. Чтобы выполнить это условие, преподаватель должен посмотреть фильм и решить, какому уровню он соответствует. Однако это требует больших временных затрат.

**Заказчику необходимо**: 
- разработать ML решение для автоматического определения уровня сложности англоязычных фильмов;
- развернуть приложение для демонстрации работы ML решения (заказчик допустил возможность использование пакета **Streamlit**).

**Исходные данные**: 
- размеченный датасет с названиями фильмов в формате *excel*, субтитрами и меткой уровня сложности языка **(A1/A2/B1/B2/C1/C2)**. Предварительно данные в таблице уже обработаны (добавлены недостающие фильмы);
- файлы субтитров в формате *.srt*, отсортированные по каталогам в соответствии с уровнем сложности. Предварительно все субтитры были перенесены в общую папку для `'/Datasets/Subtitles'` загрузки;
- словари *Oxford* (на 3000 и 5000 тыс.слов), в которых слова на английском сгруппированы по уровню сложности.

**План работы**:
- загрузка необходимых библиотек;
- загрузка и ознакомление с данными;
- предобработка данных (очистка от дубликатов, проверка наличия разметки для обучающих данных, определение исходного количества представленных данных, очистка текста субтитров, разбивка на обучающую и тестовую выборки);
- препроцессинг данных (преобразование данных для обучения модели);
- определение метрики качества;
- обучение модели с побором гиперпараметров;
- оценка модели на тестовой выборке;
- развертывание модели для демонстрации работы с использованием библиотеки **Streamlit**;
- рекомендации к улучшению проекта.

**В ходе работы было выполнено:**
1. загрузка необходимых библиотек;
2. загрузка и ознакомление с данными (загружены субтитры, фильмы с метками уровня сложности, классический словарь Oxford с 5000 словами);
3. предобработка данных (очистка от дубликатов, проверка наличия разметки для обучающих данных, определение исходного количества представленных данных, очитска текста субтитров, разбивка на обучающую и тестовую выборки);
4. препроцессинг данных, а именно преобразование текстов субтитров с компактную разреженную матрицу с помощью **CountVectorizer** с гиперпараметром **min_df=4**, чтобы исключить специфические слова для определенных фильмов, которые не несут явной языковой нагрузки.;
5. добавление дополнительных признаков- количество уникальных лемм (слов основной формы), содержащихся в каждом тексте субтитров согласно уровню сложности по классичесокму словарю **Oxford**;
6. обоснование использования метрики качества **F1-weighted**;
7. обучение модели-классификатора MultinomialNB с побором гиперпараметров на 5-и фолдах перекрестной проверки;
8. оценка модели на тестовой выборке;
9. обучение модели на всей выборке с подбором гиперпарамтера на 5-и фолдах перекрестной проверки
10. сохранение лучшей модели в формате `'joblib'`, а так же классического словаря **Oxford** для дальнейшего развертывания в демонстрационном приложении **Streamlit**.

**При решении задачи обнаружены следующие проблемы:**
1. Отсутствуют фильмы с уровнем сложности **A1**;
2. В данных присутствует сильный дисбаланс по остальным классам (меткам):
- *129 фильмов* с уровнем сложности *3* (метка **B2**);
- *56 фильмов* с уровнем сложности *2* (метка **B1**);
- *39 фильмов* с уровнем сложности *4* (метка **C1**);
- *32 фильма* с уровнем сложности *1* (метка **A2**).
3. Наличие большой вероятности в субъективности разметки данных в фильмах. Каждый специалист может немного по-разному определять уровень сложности понимания английского. В таком случае можно ожидать **смещения** в предсказании любой модели, так как субъективность часто вносит неустранимую ошибку в предиктивность модели. С большой долей вероятности модель будет допускать ошибки в предсказании. Особенно на граничных значениях меток (например ошибаться в присвоении метки **A2** или **B1**, **B1** или **B2**). Такие ошибки в принципе допустимы в отношении поставленной задачи, так как позволяют добиться того, чтобы  **студент понимал 50-70 % диалогов** даже при ошибочном предсказании модели в сторону присвоения более низкой метки, так как различия между такими классами **менее существенны**.

**Выводы по результатам работы модели:**

Лучшая модель показала результат - 0.63 (**F1_weighted**) на тестовой выборке при подобранном гиперпараметре `'alpha'` равном **0.137**. 

- как и ожидалось, модель чаще всего ошибается на граничных метках. Например модель часто назначает фильмам с меткой **B2** метку **B1**. По условиям задачи где, студенты должны минимум понимать 50-70% диалогов, ошибка прогнозирования в сторону наименьшего класса допустима в пределах граничных классов;
- хуже всего классифицированы фильмы с метками **A1** и **B1**.  В первом случае за счет низкого значения метрики **recall** (среди небольшого количества фильмов представленных в этом классе меток (7 фильмов), всего 2 фильма размечены верно). Во втором случае за счет низкого значения метрики **precision** (модель часто ошибается в присвоении метки **B1**, т.е. среди всех размеченных фильмов с присвоением метки **B1** (21 фильм) только 8 размечены верно.

Использование дополнительно созданных признаков, с количеством уникальных слов из классического словаря Oxford в каждом фильме, не дали значительного прироста в качестве предсказаний, что может подтверждать предположение о субъективности разметки представленных Заказчиком фильмов, в силу предвзятости специалистов по английскому языку в отношении сложности диалогов.

В целом с учетом предположения о субъективности разметки представленных Заказчиком фильмов, а так же использования быстрого и простого наивного Байесовского классификатора, удалось достичь приемлемых результатов. Взвешенная метрика **F1-weighted** на модели, обученной на всех представленных данных, при кроссвалидации на 5-и фолдах показала результат в **0.68** по метрике **F1-weighted**.

**Рекомендации:**
1. Собрать больше фильмов с размеченным уровнем сложности с одинаковым количеством в каждом классе, включая уровень **A1**. В данном случае можно добиться лучшей прогностической способности модели;
2. Разметка фильмов по уровню сложности должна быть лишена субъективности и опираться на общепринятые правила определения. Размеченные данные (фильмы) должны поступать от классифицированных специалистов по английскому языку, либо с общеизвестных и общепринятых источников;
3. (**Предпочтительная и важная рекомендация для Заказчика**) Для создания модели предсказания уровня сложности понимания фильмов необходимо **полностью** исключить заранее размеченные данные. 
В таком случае обучение модели будет основываться на других правилах, таких как: 
- классический и американский словари Oxford, 
- количество слов в предложениях, 
- скорость речи, 
- центральные оценки (среднее, медиана) количества букв в слове в общем по всему тексту 
- и т.д. 

В данном случае снижен риск субъективности в оценке сложности понимания английских фильмов, так как будут использованы более понятные и строгие правила определения.
