## Термины и определения

* Корутина - объект, результат вызова функции определенной с помощью конструкции async def. 
При вызове асинхронной функции не начинается её выполнение, а лишь создается корутина, которую в дальнейшем необходимо запустить на исполнение.


## Как запускать корутины

В asyncio предусмотрено несколько способов для запуска корутин:
#### 1. asyncio.run()

Является точкой входа в асинхронную программу, её следует использовать только один раз.
Вызов данной функции создает новый цикл событий и при завершении закрывает его, 
поэтому вызвать её внутри другого запущенного цикла событий не выйдет.

#### 2. Простой await корутин
```commandline
async def func(delay, func_name):
    print(f'Start {func_name}')
    await asyncio.sleep(delay)
    print(f'Complete {func_name}')

async def main():
    print(f"Start {time.strftime('%X')}")
    await func(2, 'func-1')
    await func(4, 'func-2')
    print(f"Finish {time.strftime('%X')}")

asyncio.run(main())

# Start 03:50:39
# Start func-1
# Complete func-1
# Start func-2
# Complete func-2
# Finish 03:50:45
```
В данном случае обе функции выполняются последовательно (время выполнения - 6 с.), эвейт корутины 
(эвейтится именно корутина, корутина это результат вызова func()),
исполняет корутину и дожидается окончания её выполнения и только тогда переходит на следующую строку и эвейтит следующую корутину.

#### 3. asyncio.create_task()
```commandline
async def func(delay, func_name):
    print(f'Start {func_name}')
    await asyncio.sleep(delay)
    print(f'Complete {func_name}')

async def main():
    print(f"Start {time.strftime('%X')}")
    task_1 = asyncio.create_task(func(2, 'func-1'))
    task_2 = asyncio.create_task(func(4, 'func-2'))
    await task_1
    await task_2
    print(f"Finish {time.strftime('%X')}")

asyncio.run(main())

# Start 03:55:46
# Start func-1
# Start func-2
# Complete func-1
# Complete func-2
# Finish 03:55:50
```

Сразу обращаем внимание, что время выполнения программы сократилось с 6 до 4 с. (до времени наиболее длительной задачи).
Здесь, вместо эвейта корутин мы создаём задачи, которые начинают исполняться сразу же после их создания. 
Но если задачи начинают выполняться сразу, для чего тогда эвейтить объекты этих задач? 
Всё просто, эвейт задачи говорит программе дождаться её завершения, прежде чем переходить на следующую строку.

Говоря по секрету, эвейтить задачи вовсе не обязательно, можете вместо этого написать await asyncio.sleep(10)
и за эти 10 секунд задачи успеют завершиться сами (*так делать плохо, не будьте такими).
Для того чтобы дождаться выполнения нескольких задач, наиболее удачным решением будет использование функции gather 
(await asyncio.gather(task_1, task_2, task_3, ...)), подробнее об этой функции мы поговорим дальше.
Если же несколько задач выполняются бесконечно, то можно просто написать await asyncio.Future(). 

*Важное замечание: при создании задачи, на неё будет создана только слабая ссылка,
а значит что garbage collector может удалить задачу из памяти прямо в процессе выполнения.
Такое поведение никому не нужно, поэтому сохраняйте ссылки на созданные задачи, например добавляя их в список или множество.

```commandline
tasks = set()
task_1 = asyncio.create_task(func(6, 'func-1'))
tasks.add(task_1)
task_2 = asyncio.create_task(func(4, 'func-2'))
tasks.add(task_2)
await asyncio.gather(*tasks)
```

*В более ранних версиях Python использовался другой синтаксис, нужно было получить объект цикла событий, 
создать в нём задачи и запустить цикл пока задачи не выполнятся. 
Но если вы используете версию 3.10 и выше, интерпретатор  сам выведет вам предупреждение DeprecationWarning, 
поэтому пользуйтесь новым синтаксисом ;)

```commandline
print(f"Start {time.strftime('%X')}")
loop = asyncio.get_event_loop()
task1 = loop.create_task(func(2, 'func-1'))
task2 = loop.create_task(func(4, 'func-2'))
loop.run_until_complete(asyncio.wait([task1, task2]))
print(f"Finish {time.strftime('%X')}")

# DeprecationWarning: There is no current event loop
```

#### 4. asyncion.TaskGroup (начиная с python 3.11)
```commandline
async def func(delay, func_name):
    print(f'Start {func_name}')
    await asyncio.sleep(delay)
    print(f'Complete {func_name}')

async def main():
    async with asyncio.TaskGroup() as tg:
        tg.create_task(func(2, 'func-1'))
        tg.create_task(func(4, 'func-2'))
        print(f"Start {time.strftime('%X')}")
    print(f"Finish {time.strftime('%X')}")

asyncio.run(main())
```
Это современная альтернатива asyncio.create_task(). 
Это удобный и надежный вариант дождаться завершения выполнения всех задач в группе.