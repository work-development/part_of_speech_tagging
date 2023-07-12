#  Определение частей речи слов    

<p align="justify">
Будем использовать размеченный корпус, который называется "SynTag Rus". В корпусе содержится разметка по частям речи, по нормальным формам слов, синтаксическая разметка.  Решать задачу определения части речи мы будем с помощью свёрточных нейросетей, причём будем использовать нейросети, которые принимают на вход номера отдельных символов    
</p>

## Модель     

* Cвёрточная нейросеть

<p align="justify">
Составим словарь символов (отображение символа в его номер).Чтобы уравнять длины всех токенов и всех предложений введем пустой символ. Аналогичную процедуру проделаем и с метками частей речи. Затем перекладываем весь исходный корпус  в структуры pytorch Dataset. Для этого в TensorDataset подаем два тензора: первый — это тензор идентификаторов символов, а второй — это идентификаторы меток частей речи.
</p>

##  Универсальные компоненты   


## Основные функции и классы   

<p align="justify">
<b>pos_corpus_to_tensor</b> -  на вход получает список токенизированных предложений в котором токены имеют информацию о частях речи (т.к. формат CoNLL). Эта функция предназначена для преобразования обучающей и тестовой выборки в 2 тензора — тензор входных идентификаторов символов и тензор выходных идентификаторов тэгов. Тензор номеров символов имеет следующую размерность: [количество предложений (всего в обучающей выборке) на количество токенов в предложений на максимальную длину токена]. Но, кроме этого, мы добавляем ещё 2 дополнительных колоночки к каждому токену. Эти колоночки дополнительные мы будем заполнять нулями — они нам нужны для того, чтобы указать нейросети, что определённая N-грамма символов встречается именно в начале токена или именно в конце токена, но не в середине (чтобы нейросеть умела отличать начало слова и конец слова от середины слова).
</p>

<p align="justify">
<b>StackedConv1d</b> - свёрточный модуль. В основе модуля лежит набор блоков, связанных между собой при помощи skip connection. Каждый из этих блоков реализуется с помощью модуля pytorch "sequential" — это базовый модуль, который берёт список других модулей и выполняет их по-очереди. В этом блоке первый слой — это одномерный свёрточный слой. Чтобы размерность тензора не менялась перед тем, как применять свёртки, добавим какое-то количество нулей в начало и в конец тензора (padding). При этом сверточный слой не меняет число каналов. Второй слой в нашем блоке — это dropout (чтобы нейросеть меньше переобучалась). Третий слой нашего блока — это функция активации (Leaky ReLU).
</p>

<p align="justify">
<b>SingleTokenPOSTagger</b> - модуль, предсказывающий метки частей речи токенов, используя информацию, содержащуюся только в самих токенах.
</p>
