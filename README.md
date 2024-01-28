##### <a name="pageup"></a>

# 1CBPListOfDocs

🗺️ В 1С Бухгалтерия есть отчет "Реестр документов" цель которого вывести на печать любую форму списка,
данный репозиторий содержит расширение которое добавляет команду печать "Реестр документов" нужной вам форме списка,
а также расширение которое заменить подписанта для указанного объекта в печатной форме "Реестр документов".

👀 ["Видео инструкиця"](https://youtu.be/)

📜 Если вы создали объект или для типового объекта которому надо добавить команду печать "Реестр документов"
или когда вам необходимо заполнять подпись конкретных документов не главным бухгалтером (изменить типовое поведение).

# Разделы

+ [Внедрение подключения команды печать](#step0); ➕
+ [Команда печать для не типового объекта](#step1);  🟣
+ [Внедрение подписи ответственного лица](#step2); 🔘

##### <a name="step0"></a> Внедрение подключения команды печать [(начало)](#pageup);

1. Создаем расширение.
2. Загружаем в него расширение данного репозитория "КомандаПечатьРеестрДокументов.cfe".
3. Открываем общий модуль "ПриОпределенииОбъектовСКомандамиПечати" и добавляем целевой объект.
4. В менеджере целевого объекта переопределяем процедуру "ДобавитьКомандыПечати", вызов "Перед"
добавляем блок кода и задаем "Представление" + "ЗаголовокФормы"
 ```
	// Реестр документов
	КомандаПечати = КомандыПечати.Добавить();
	КомандаПечати.Идентификатор  = "Реестр";
	КомандаПечати.Представление  = НСтр("ru = 'Реестр документов'");
	КомандаПечати.ЗаголовокФормы = НСтр("ru = 'Реестр документов ""Реализация отгруженных товаров""'");
	КомандаПечати.Обработчик     = "УправлениеПечатьюБПКлиент.ВыполнитьКомандуПечатиРеестраДокументов";
	КомандаПечати.СписокФорм     = "ФормаСписка";
	КомандаПечати.Порядок        = 100;

```
В модуле формы списка целевого объекта:
1. переопределяем процедуру "ПриСозданииНаСервереПеред" и добавим код:
 ```
АдресХранилищаНастройкиДинСпискаДляРеестра = ПоместитьВоВременноеХранилище(Неопределено, УникальныйИдентификатор);
```
2. Создаем на форме реквизит "АдресХранилищаНастройкиДинСпискаДляРеестра" тип строка, неограниченной длины.
3. В модуле формы добавляем процедуру:
 ```
&НаСервере
Процедура НастройкиДинамическогоСписка()	
	Отчеты.РеестрДокументов.НастройкиДинамическогоСписка(ЭтотОбъект);	
КонецПроцедуры
 ```
4. переопределяем процедуру "Подключаемый_ВыполнитьКоманду" вызов "Вместо с контролем" и добавим вставку:
 ```
// в самом начале процедуры
	#Вставка
	Если Команда.Имя = "ПодменюПечатьОбычное_Реестр" Тогда
		НастройкиДинамическогоСписка();
	КонецЕсли;
	#КонецВставки
 ```

##### <a name="step1"></a> Команда печать для не типового объекта [(начало)](#pageup);

Выполните все как для типового, а пункт 4 (работа с формой списка) следуйте этой инструкции:
1. создайте реквизит на форме
2. Добавьте в модуль код (на забудьте связать событие формы "ПриСозданииНаСервере"):
 ```
#Область ОбработчикиСобытийФормы

&НаСервере
Процедура ПриСозданииНаСервере(Отказ, СтандартнаяОбработка)

	// СтандартныеПодсистемы.ПодключаемыеКоманды
	ПодключаемыеКоманды.ПриСозданииНаСервере(ЭтотОбъект);
	// Конец СтандартныеПодсистемы.ПодключаемыеКоманды

	АдресХранилищаНастройкиДинСпискаДляРеестра = 
	ПоместитьВоВременноеХранилище(Неопределено, УникальныйИдентификатор);
	
КонецПроцедуры

#КонецОбласти

#Область СлужебныеПроцедурыИФункции

&НаСервере
Процедура НастройкиДинамическогоСписка()
	
	Отчеты.РеестрДокументов.НастройкиДинамическогоСписка(ЭтотОбъект);
	
КонецПроцедуры

#КонецОбласти

#Область СлужебныеПроцедурыИФункцииБСП

// СтандартныеПодсистемы.ПодключаемыеКоманды
&НаКлиенте
Процедура Подключаемый_ВыполнитьКоманду(Команда)
	Если Команда.Имя = "ПодменюПечатьОбычное_Реестр" Тогда
		НастройкиДинамическогоСписка();
	КонецЕсли;
	ПодключаемыеКомандыКлиент.ВыполнитьКоманду(ЭтотОбъект, Команда, Элементы.Список);
КонецПроцедуры

&НаСервере
Процедура Подключаемый_ВыполнитьКомандуНаСервере(Контекст, Результат)
	ПодключаемыеКоманды.ВыполнитьКоманду(ЭтотОбъект, Контекст, Элементы.Список, Результат);
КонецПроцедуры

&НаКлиенте
Процедура Подключаемый_ОбновитьКоманды()
	ПодключаемыеКомандыКлиентСервер.ОбновитьКоманды(ЭтотОбъект, Элементы.Список);
КонецПроцедуры
// Конец СтандартныеПодсистемы.ПодключаемыеКоманды

#КонецОбласти

 ```

##### <a name="step2"></a> Пример задачи [(начало)](#pageup);

Пользователи которые не входят в группуДоступа "РасширенныйДоступСправочникНоменклатура"
должны получить доступ на редактирование исключительно к реквизиту Наименование и команда "Записать и закрыть"
, а те пользователи которые входят в группу "РасширенныйДоступСправочникНоменклатура" к ним ограничения применятся не должны!
