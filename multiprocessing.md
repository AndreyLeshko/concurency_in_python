# multiprocessing

Начнём знакомиться с мультипроцессностью (Не мультипроцессорность! Не путайте понятия.).
Разберем для начала простенький пример:

```commandline
    from multiprocessing import Process
    
    def process_func(process_num):
        print(f'Hello from process {process_num}')
    
    if __name__ == '__main__':
        p1 = Process(target=process_func, args=(1,), daemon=True)
        p2 = Process(target=process_func, args=(2,), daemon=True)
        p1.start()
        p2.start()
        p1.join()
        p2.join()
```

