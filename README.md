# Лабораторная 3: Bison & ANTLR
Вариант: 3 (Перевод с Python на Си)

## Грамматика
В грамматику входят обычные арифметические выражения, присваивания, `for` и `if`
блоки а также `def`-блок (TODO: возможно стоит убрать).


FIXME: у бизоновых правил есть приоритет (точно не помню какой), поэтому если я
неправильно сочиню грамматику, возможны проблемы.
```
file:
    statements EOF

statements:
    statement statements
    | statement

statement:
    compound_stmt
    | simple_stmt

// empty lines should not contain any symbols (even ws), otherwise it's an error
simple_stmt:
    assignment NEWLINE
    | NEWLINE

assignment:
    IDENTIFIER EQ_SIGN expr

// expr in `if` must be of type int
// expr in `if` must be of special `range` type
compound_stmt:
    IF_KW expr COLON_SIGN NEWLINE INDENT statements DEDENT
    | FOR_KW IDENTIFIER IN_KW expr COLON_SIGN NEWLINE INDENT statements DEDENT

// all expressions will be typed and the types will be inferred during semantic
// analysis

expr:
    expr '+' term
    | term

term:
    term '*' factor
    | factor

factor:
    NUMBER
    | IDENTIFIER LPAREN expr RPAREN
    | IDENTIFIER


```


## Лексический анализатор (токенайзер)
Для определения вложенности я решил ввести необычный токен - `DEDENT`. Его можно
увидеть в грамматике из оффициальной доки питона: https://docs.python.org/3/reference/grammar.html

Ещё больше информации о DEDENT здесь: https://docs.python.org/3/reference/lexical_analysis.html#indentation

Другого решения для определения вложенности я не могу придумать, кажется что без
этого никак.

Все отступы по умолчанию равны четырём пробелам, я решил не мудрить с разными
величинами отступов. Табы тоже не принимаются за отступы.

Список лексем (токенов):
```
Токен | Описание
IDENTIFIER | идентификатор (переменная/функция)
CONST | Численная константа

```


++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
+                                  Мои мысли                                   +
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

Итак, щас буду писать как я разбираюсь в бизоне, это не должно попасть в
финальный отчет.
- Semantic value - это например конкретное значение "245" - по сути это токен
  NUMBER, но имеет semantic value равное 245.
- Semantic actions - это как мы генерируем semantic value по правилу (например,
  когда парсим "2 + 4").
- GLR - когда возникает конфликты r/r или s/r, то он ветвится. Если какая-то
  ветка фейлится, то она отбрасывается. Важно что при этом semantic values не
  вычисляются а только "записываются" их вычисления, которые отыгрываются как
  только остается одна ветка исполнения.
- 1.8 описывает процесс создания парсера. Оказывается лексический анализатор
  (токенайзер) я должен писать сам((( Вроде можно сгенерить с помощью Lex
- Презентация про токенайзеры: https://ii.uni.wroc.pl/~nivelle/teaching/compilers2013/tokenizer.pdf
- Остановился на 3 - Bison Grammar Files (page 47)
- Похоже мне нужно почитать 10 - Parsers Written in Other Languages.
- Могут возникнуть проблемы с динамической типизацией - на этапе семантического
  анализа надо будет делать infer типа переменной по присваиваемому значению и
  если последующие присвоения не согласуются по типам, то будем докладывать
  ошибку. Btw, поскольку это будет уже семантический анализ, то нам хорошо бы
  хранить позиции начала и конца каждой ноды в AST.

if kek > 0:
    a = 10
    b = 20
    if kek > 5:
        c = 10
        d = 20
    e = 4

Моё решение: в токенайзере добавлять токен типа DEDENT (поддерживаем уровень
отступа на текущей строке). По умолчанию, отступы - 4 пробела. Любое другое
число я запрещаю использовать, потому что это ёбнешься парсить. Табы не хочу
поддерживать.

