
## ДЕКОРАТОРЫ
	
 - Что это?
 - Зачем нужно?
 - Структура
 - Использование
 - Примеры



## Что это?
 *Обертка другой функции, которая позволяет выполнять произвольный код, внутри функции (динамическое подключение дополнительного поведения к объекту)*

## Зачем нужно?

Для выноса повторяющегося кода, который не относится к целевому коду функции.
 
## Структура

*2 или 3 Элемента.
2 элемента - когда у декоратора нет аргументов
3 - когда есть.*

    3 элемента
    def decorator(*args, **kwargs):
    	def output(initial_func):
       		def wrapper(*argses, **kwargses):
    			result = initial_func(*argses, **kwargses)
    			return result
    		return wrapper
    	return output




# Использование
 

## Ручная запись


    variable_name = decorator(something, 'something', some=1)(initial_func)(arguments_of_initial_func)

 

## Сахар



    @decorator(something, 'something', some=1)
    def initial_func(*args, **kwargs)
    	pass


## Пишем декоратор

**
Исключаем dry, пишем декоратор, который буде показывать время работы функции*
Проверка скорости работы генератора и иттератора

    # -*- coding: utf-8 -*-
    from datetime import datetime
    
    def timing(*argses, **kwargs):
    	"""
    	Декоратор, который показывает время выполнения функции
    	"""
    
    	def outing_wrapper(function):
    		print(f'Ваша функция обернута с именем {kwargs.get("name")}')
    		def wrapper(*args, **kwargs):
    			"""
    			Функция обертка, будет выполнять произвольный код внутри другой(оборачиваемой функции)
    			*args и **kwargs для отслеживания аргументов в исходной функции
    			"""
    			time = datetime.now()
    			result = function(*args, **kwargs)
    			time_end = datetime.now() - time
    			print(time_end)
    			return result
    		return wrapper
    	return outing_wrapper

## Декорируем функцию


    @timing(name='Iterator')
    def iterator(n):
    	"""
    	Функция возвращает список с четными значениями через иттератор
    	"""
    	l = []
    	for i in range(n):
    		if i % 2 == 0:
    			l.append(i)
    	return l 
    
    @timing(name='Generator')
    def generator(n):
    	"""
    	Функция возвращает список с четными значениями через генератор
    	"""
    	l = [x for x in range(n) if x % 2 == 0]
    	return l



    list_gen = generator(10)
    print(list_gen)
    #Ваша функция обернута с именем Generator
    #0:00:00.000015
    #[0, 2, 4, 6, 8]
    
    #Тоже самое вручную
    list_gen_man = timing(name='Generator')(generator)(10)
    print(list_gen_man)







