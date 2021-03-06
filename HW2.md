## Домашнее задание №2

Дедлайн: 23 марта.

#### 1 Монадический стек RWS
 __# 1.1 Reader__

Зафиксируем некоторый тип `r`, заметим, что функции вида `r -> a` являются функтором, действительно:

```haskell
instance Functor ((->) r) where
  fmap = (.)
```

Поскольку инстанс `Functor` для `((->) r` определен в `GHC.Base`, воспользуемся типом-обёрткой:

```haskell
newtype Reader r a = Reader { runReader :: r -> a }
```

Семантика этого типа такова: вычисления, которые происходят в некотором общем окружении `r`, которое можно локально изменять.

При работе с монадой `Reader r` удобно использовать следующие функции:

1. `ask` - возвращает окружение

2. `local` --- выполняет вычисление в модифицированном окружении.


__Задача #1.1.1__: реализуйте инстансы `Functor`, `Applicative` и `Monad` для `Reader r`. Использование `deriving` в каком-либо виде запрещено.

__Задача #1.1.2__: реализуйте функции-помощники `ask`, `local`.

```haskell
ask :: Reader r r
ask = _

local
  :: (r -> r)
  -> Reader r a
  -> Reader r a
local = _

instance Functor (Reader r) where
instance Applicative (Reader r) where
instance Monad (Reader r) where
```

__#1.2 Writer__

Семантика этого типа такова: Writer является оберткой над типом упорядоченной пары, первым элементом которой является некоторый результат вычисления, а вторым - лог для актуального результата вычислений.

__Задача #1.2.1__: реализуйте инстансы `Functor`, `Applicative` и `Monad` для `Writer w`. Использование `deriving` в каком-либо виде запрещено.

При работе с монадой `Writer w` удобно использовать следующие функции:

1. `tell` - записывает значение в `Writer`.

2. `listen` - функция, заменяющая внутреннее состояние.

3. `pass` - функция, изменяющая лог, но при этом сохраняет значение.

__Задача #1.2.2__: реализуйте функции-помощники @tell@, @listen@ и @pass@.

```haskell
newtype Writer w a
  = Writer { runWriter :: (a, w) }

tell
  :: Monoid w
  => w
  -> Writer w ()
tell = _

listen
  :: Monoid w
  => Writer w a
  -> Writer w (w, a)
listen = _

pass
  :: Monoid w
  => Writer w (a, w -> w)
  -> Writer w a
pass = _

instance Functor (Writer w) where
instance Monoid w => Applicative (Writer w) where
instance Monoid w => Monad (Writer w) where
```

__#1.3 State__

Часто бывает так, что нужно использовать состояние, которых, как известно, в Haskell нет.

Для эмуляции состояния принято использовать монаду `State s`.

__Задача #1.3.1__: реализуйте инстансы `Functor`, `Applicative` и `Monad` для `State s`. Использование `deriving` в каком-либо виде запрещено.

При работе с монадой `State s` удобно использовать следующие функции:

1. `get :: State s a` --- функция, возвращающая внутреннее состояние,

2. `put :: s -> State s ()` --- функция, заменяющая внутреннее состояние.

__Задача #1.3.2__: реализуйте функции-помощники `get`, `put`.

```haskell
newtype State s a
  = State { runState :: s -> (a, s) }

get :: State s s
get = _

put :: s -> State s ()
put = _

instance Functor (State s) where
instance Applicative (State s) where
instance Monad (State s) where
```

#### 2. Задачи на стрелки

2.1. Реализовать категорию стрелок, хранящую последний вход и время прошедшее с последнего входа (как `Double`), и инстансы `Category`, `Arrow`. Определение стрелок вы найдёте в модуле `Control.Arrow` в `base`. Рекомендация: адаптируйте монаду State. Поизучайте первый слитый вариант задания.

```haskell
data SignalFunction a b
instance Category SignalFunction
instance Arrow SignalFunction
```


2.2 Реализовать стрелку, интегрирующую ломаную с узлами (t_i, x_i).

```haskell
integral :: SignalFunction Double Double
integral = _
```

2.3 Используя расширение -XArrows и arrow-нотацию реализовать схему:

```
     +=========+
---- |   * 2   | ---
     +=========+    \   +=========+        +==========+
                     -- |    +    | ------ | integral | -----
     +=========+    /   +=========+ \      +==========+
---- |   * 3   | ---                 ------------------------
     +=========+
```

Эта схема принимает `(x, y)`, а возвращает `(I, 2x + 3y)`, где `I` - накопленная сумма под графиком ломанной.  Все вычисления должны проходить в стрелках (не должно быть let-присваиваний и  функций справа от -<)

```haskell
someFunction :: SignalFunction (Double, Double) (Double, Double)
someFunction = _
```


2.4 Написать функцию, запускающую схему на списке входных данных и временных дельтами

```haskell
runSignalFunction someFunction (0, 0) [ ((1, 2), 0.1)
                                      , ((3, 4), 0.2)
                                      , ((5, 6), 0.3) ] =
                                      [ (0.4, 8)
                                      , (3, 18)
                                      , (9.9, 28) ]
```

Аргументы - стрелка, значение в момент времени 0, список входных данных, сопровождаемых временной дельтой.
```haskell
runSignalFunction :: SignalFunction a b -> a -> [(a, Double)] -> [b]
runSignalFunction sf atZero inputs = outputs
  where
  outputs = _
```

#### №3 Задачи на Concurrency

Используя библиотеку STM, и модули Control.Concurrent, Conctrol.Exception из base
решить задачу "Обедающих философов".

У нас есть 5 философов, представляемые отдельными потоками, и 5 вилок между ними, которые. Каждый философов может размышлять или обедать, его желания переключаются отправкой сигнала в поток. Если философ хочет есть и рядом с ним есть две свободные вилки, он берёт их в руки и ест. Необходимо добиться, чтобы философы не блокировали другу друга.

Пример блокировки:
1. Первый философ ест
2. Второй философ проголодался, но рядом всего одна свободная вилка. Он берёт её в руки
и есть
3. Третий философ проголодался, но второй сам не ест и ему не даёт.

Пример dead-lock'а:
1. Каждый философ берёт вилку, а потом ещё одну вилку. Из-за несогласованности
получилось, что каждый философ оказался с одной вилкой в руках и никто не может есть

3.1. Написать необходимые типы. Состояние храним с помощью монады STM.

```haskell
data Fork = ForkAcquired | ForkReleased
data Forks -- = ?
```

Сигнал переключающий философа
```haskell
data PhilosopherSignal = Hungry | Full
instance Exception PhilosopherSignal
```

Любая специфичная для конкретного философа информация.
```haskell
data PhilosopherState = ?
```

3.2. Написать базовый функционал.

```Haskell
-- Отправляем сигналы
makeHungry :: ThreadId -> IO ()
makeHungry = _

makeFull :: ThreadId -> IO ()
makeFull = _
```

Первый аргумент - часть общего состояния. Философ должен обрабатывать исключения типа PhilosopherSignal и переключать своё поведение.
```haskell
philosopher :: PhilosopherState -> IO ()
philosopher = _
```

Функция возвращающая ссылки на потоки.
```haskell
makePhilosophers :: IO [ThreadId]
makePhilosophers = _
```

#### №4 Задача на continuation passing style

Рассмотрим тип `MyCont r a`, параметризованный типами `r` и `a`. Данный тип - обертка над типов функции `(a -> r) -> r`. Что в функциональном программировании еще часто называется монадой `Cont`, предназначенной для так называемых функций с продолжением.

```haskell
newtype MyCont r a
  = MyCont { runCont :: (a -> r) -> r }
```

4.1 Реализовать инстанс функтора для типа MyCont r
```haskell
instance Functor (MyCont r) where
```

4.2 Реализовать инстанс аппликатива для типа MyCont r

```haskell
instance Applicative (MyCont r) where
```

4.3 Реализовать инстанс монады для типа MyCont r

```haskell
instance Monad (MyCont r) where
```

#### №5 Трасформеры монад

Рассмотрим класс `MonadTrans`. MonadTrans позволяет делать новую монаду из существующей монады, вкладывая в новую монаду все вычисления и действия из предыдущей монады. Такой способ формирования новых монад называется трансформером монад,
и задается классом `MonadTrans`:

```haskell
class MonadTrans n where
  lift :: Monad m => m a -> n m a
```
MonadTrans -- это класс с единственным методом, который берет значение в монаде `m` и посылает его в новую монаду `n`.

Реализовать инстанс `MonadTrans` для следующих типов

5.1. `MaybeT`

```haskell
newtype MaybeT m a
  = MaybeT { runMaybeT :: m (Maybe a) }

instance MonadTrans MaybeT where
```

5.2. ContT

```haskell
newtype ContT r m a
  = ContT { runContT :: (a -> m r) -> m r }

instance MonadTrans (ContT r) where
```

5.3. StateT

```haskell
newtype StateT s m a
  = StateT { runStateT :: s -> m (a, s) }

instance MonadTrans (StateT s) where
```
