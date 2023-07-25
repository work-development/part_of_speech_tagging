#  Определение частей речи слов    

<p align="justify">
Будем использовать размеченный корпус, который называется "SynTag Rus". В корпусе содержится разметка по частям речи, по нормальным формам слов, синтаксическая разметка. Решать задачу определения части речи мы будем с помощью свёрточных нейросетей, причём будем использовать нейросети, которые принимают на вход номера отдельных символов.   
</p>

## Модель     

* Cвёрточная нейросеть (простенький ResNet)


<p align="justify">
Составим словарь символов (отображение символа в его номер).Чтобы уравнять длины всех токенов и всех предложений введем пустой символ. Аналогичную процедуру проделаем и с метками частей речи. Затем перекладываем весь исходный корпус  в структуры pytorch Dataset. Для этого в TensorDataset подаем два тензора: первый — это тензор идентификаторов символов, а второй — это идентификаторы меток частей речи.
</p>

<p align="justify">
Предсказание частей речи на уровне предложений (с учетом контекста).
</p>

##  Универсальные компоненты  
 * N-граммы
 * Cross entropy


## Основные функции и классы   

<p align="justify">
<b>pos_corpus_to_tensor</b> -  на вход получает список токенизированных предложений в котором токены имеют информацию о частях речи (т.к. формат CoNLL). Эта функция предназначена для преобразования обучающей и тестовой выборки в 2 тензора — тензор входных идентификаторов символов и тензор выходных идентификаторов тэгов. Тензор номеров символов имеет следующую размерность: [количество предложений (всего в обучающей выборке) на количество токенов в предложений на максимальную длину токена]. Но, кроме этого, мы добавляем ещё 2 дополнительных колоночки к каждому токену. Эти колоночки дополнительные мы будем заполнять нулями — они нам нужны для того, чтобы указать нейросети, что определённая N-грамма символов встречается именно в начале токена или именно в конце токена, но не в середине (чтобы нейросеть умела отличать начало слова и конец слова от середины слова).
</p>

<p align="justify">
<b>StackedConv1d</b> - свёрточный модуль. В основе модуля лежит набор блоков, связанных между собой при помощи skip connection. Каждый из этих блоков реализуется с помощью модуля pytorch "sequential" — это базовый модуль, который берёт список других модулей и выполняет их по-очереди. В этом блоке первый слой — это одномерный свёрточный слой. Чтобы размерность тензора не менялась перед тем, как применять свёртки, добавим какое-то количество нулей в начало и в конец тензора (padding). При этом сверточный слой не меняет число каналов. Второй слой в нашем блоке — это dropout (чтобы нейросеть меньше переобучалась). Третий слой нашего блока — это функция активации (Leaky ReLU).
</p>

<p align="justify">
<b>SingleTokenPOSTagger</b> - модуль, предсказывающий метки частей речи токенов, используя информацию, содержащуюся только в самих токенах. Для каждого символа получаем вектора (инициализируемые случайными числами). Затем эти вектора мы передаём в свёрточный модуль, для того, чтобы учесть локальный контекст. Затем используем max pooling для агрегации признаков символов в токене, чтобы получить вектор токена. Для того чтобы по каждому токену принять решение касательно его метки, признаки токенов мы передаём в ещё один нейросетевой модуль, который называется out — это просто полносвязный блок. В результате применения этого блока мы получаем также двухмерный тензор, но у него уже размер строки - "количество меток частей речи". Благодаря "skip connections", N-граммы, которые учитываются этой нейросетью, по сути, имеют различную длину. Например, если мы используем размер ядра свёртки, равный 3, то первый блок учитывает трёхграммы, второй блок уже учитывает пятиграммы, а третий — семиграммы, соответственно. При этом, благодаря тому, что есть "skip connection", информация о трёхграммах не теряется, она пробрасывать до самого конца.
</p>

<p align="justify">
<b>SentenceLevelPOSTagger</b> - функция учитывающая контекст для учета слов "с разной частью речи" пишущихся одинаково.  Эта функция подает векторы символов в свёрточную нейросеть "single token backbone". Таким образом, получаем векторы символов с учётом их контекста. Затем используем глобальный пулинг для того, чтобы получить вектор токена. Далее делаем тоже самое но уже на уровне токенов в предложении. Далее тензор содержащий признаки токенов без учёта их контекста подаётся в другой свёрточный модуль для того, чтобы учесть контекст.

</p>
