#HSLIDE

## OTP.
## GenServer, Supervisor, Application

#HSLIDE

### Преговор
Какво правихте през последния 1 месец?

#HSLIDE

### Open Telecom Platform
* Какво НЕ е OTP <!-- .element: class="fragment" -->
* Kакво е OTP <!-- .element: class="fragment" -->

#HSLIDE

### Какво НЕ е OTP
* OTP не се използва само за писане на телеком програми <!-- .element: class="fragment" -->
* OTP не е нещо отделно от Erlang. Дистрибуцията се нарича Erlang/OTP, а не Erlang + sometimes OTP. <!-- .element: class="fragment" -->
* Github организацията се нарича erlang, а репото - otp  <!-- .element: class="fragment" -->
* OTP не е само стандартната библиотека на Erlang <!-- .element: class="fragment" -->

#HSLIDE
### Какво е OTP
* Интерпретатор и компилатор на Erlang <!-- .element: class="fragment" -->
* Стандартните библиотеки на Erlang <!-- .element: class="fragment" -->
* Dialyzer, за който говорихме в Тип-спецификации и поведения <!-- .element: class="fragment" -->
* Mnesia - дистрибутирана база данни <!-- .element: class="fragment" -->
* ETS - база данни в паметта <!-- .element: class="fragment" -->
* Дебъгер <!-- .element: class="fragment" -->
* И много други… <!-- .element: class="fragment" -->

#HSLIDE
### Какво е OTP
В книгата **Designing for Scalability with Erlang/OTP** авторите *Francesco Cesarini* и *Steve Vinoski* дефинират OTP като три ключови компонента, които взаимодействат помежду си:

1. Самият Erlang
2. Множество от библиотеки и виртуалната машина
3. Множество от **system design principles** (принципи за дизайн на системи <sub><span style="color: #e0ebeb">некадърен превод на автора</span></sub>)

#HSLIDE
### OTP Compliant Proccess
След OTP лекциите е силно препоръчително да спрете да спрете създавате процеси чрез **spawn**. Също така всички ваши процеси трябва да са OTP съвместими. Това ще им позволява:
1. Да бъдат използвани в супервайзор дърво
2. Грешките в тези процеси да бъдат записвани с повече детайли

#HSLIDE
Но няма често да ви се налага ръчно да имплементирате OTP-compliant частта.

Erlang/OTP идва с абстракции, които имплементират OTP-съвместими процеси.

#HSLIDE
![Image-Absolute](assets/common-pattern.png)

#HSLIDE
## Welcome the GenServer

#HSLIDE
## Demo + [link](http://learnyousomeerlang.com/what-is-otp#its-the-open-telecom-platform)

#HSLIDE
Нека да разгледаме един по-прост пример за GenServer, който конвертира стрингове към числа.
```elixir
defmodule StringToInt do
  use GenServer

  def start_link() do
    GenServer.start_link(__MODULE__, :ok, nil)
  end

  def handle_call(string, _from, state) do
    result = String.to_integer(string)
    {:reply, result, state}
  end
end
```

#HSLIDE
```
iex(1)> {:ok, pid} = StringToInt.start_link()
{:ok, #PID<0.119.0>}
iex(2)> Process.alive?(pid)
true
iex(3)> GenServer.call(pid, "100")
100
iex(4)> GenServer.call(pid, "100s")
** (EXIT from #PID<0.117.0>) shell process exited with reason: an exception was raised:
    ** (ArgumentError) argument error
        :erlang.binary_to_integer("100s")
        (string_to_int) lib/string_to_int.ex:9: StringToInt.handle_call/3
        (stdlib) gen_server.erl:636: :gen_server.try_handle_call/4
        (stdlib) gen_server.erl:665: :gen_server.handle_msg/6
        (stdlib) proc_lib.erl:247: :proc_lib.init_p_do_apply/3
```

#HSLIDE
![Image-Absolute](assets/sad-panda.jpg)

#HSLIDE

#### Genius Idea!

```elixir
def handle_call(string, _from, state) do
  try do
    result = String.to_integer(string)
    {:reply, result, state}
  rescue
    error -> {:reply, "Cannot convert #{string} to integer. Reason: #{inspect(error)}", state}
  end
end
```

#HSLIDE
#### "Genius" Idea!

```elixir
def handle_call(string, _from, state) do
  try do
    result = String.to_integer(string)
    {:reply, result, state}
  rescue
    error -> {:reply, "Cannot convert #{string} to integer. Reason: #{inspect(error)}", state}
  end
end
```

#HSLIDE

Elixir Course Team
![Image-Absolute](assets/elixir-team-dissapointed.png)

#HSLIDE
## Welcome the Supervisor