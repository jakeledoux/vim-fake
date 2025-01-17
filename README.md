# Vim-Fake

Vim-fake is a vim plugin to provide a generator of random dummy/filler text,
which is useful to generate a nonsense text like [_Lorem ipsum_](https://en.wikipedia.org/wiki/Lorem_ipsum).

```vim
:echo fake#gen("paragraph")

Ignota autem nostrud albucius sagittis no fabulas erat eam ridens
et per moderatius 98 movet? Sea iure est integre metus adipisci
justo id con ultrices omnes minim odioaccommodare. Consul omnium
enim volumus lectus habeo fuisset pri utinam sem tamquam no.
```

## Usage

### Generate

`fake#gen({keyname})` is to generate a random data,
where the `{keyname}` is a data source name.
For example, if you get a country name, use `"country"`.

`fake#gen()` returns a different answer for each execution.

```vim
:echo fake#gen("country")
Canada
:echo fake#gen("country")
Nigeria
:echo fake#gen("country")
Indonesia
```

The following built-in dictionaries({keyname}) are available as data sources.
The dictionary is a simple text file which is like `/usr/share/dict/words` on UNIX.
`fake#gen({keyname})` chooses and returns a random line from the {keyname} dictionary.

| {keyname}     | Contents                    | Order               |
|:--------------|:----------------------------|:--------------------|
| male_name     | Male names in the world     | Population          |
| female_name   | Female names in the world   | Population          |
| surname       | Surnames in the world       | Population          |
| country       | Country names               | Population          |
| gtld          | gTLD                        | Number of websites  |
| job           | Job names                   | &nbsp;              |
| word          | English words               | &nbsp;              |
| nonsense      | Nonsense words              | &nbsp;              |


### Replace

To replace the following `dummy` texts with real names

```xml
<ul>
	<li> dummy </li>
	<li> dummy </li>
	<li> dummy </li>
	<li> dummy </li>
</ul>
```

Type next, `:%s/dummy/\=fake#gen("male_name")/g`

```xml
<ul>
	<li> Steve </li>
	<li> Rodney </li>
	<li> Leonard </li>
	<li> Adam </li>
</ul>
```

### Replace Placeholder

To substitute multiple places with placeholder,

```xml
<user>
	<name>:male_name:</name>
	<origin>:country:</origin>
	<job>:job:</job>
</user>
<user>
	<name>:male_name:</name>
	<origin>:country:</origin>
	<job>:job:</job>
</user>
```

`:FakeSubstitute` command is to substitute each `:{keyname}:` with
`fake#gen({keyname})`, respectively.

```xml
<user>
	<name>Bill</name>
	<origin>Pakistan</origin>
	<job>Animal Trainer</job>
</user>
<user>
	<name>Willard</name>
	<origin>Bangladesh</origin>
	<job>Civil Engineer</job>
</user>
```

### Insert

To insert these fake texts in the insert-mode,
use the expression register like `<C-r>=fake#gen("male_name")`

More easily, you probably want to use with a snippet plugin.

[vim-fake & neosnippet example - Gist](https://gist.github.com/tkhren/9fbc7227c2e9d2f4319c)

![fakesnip](https://cloud.githubusercontent.com/assets/534457/13453296/34c4895a-e092-11e5-83d3-d81afee83c37.gif)

## More Generators

**You can define a new generator in various combination of the existing `{keyname}s`.**

`fake#define({keyname}, {code})` defines a new `{keyname}`,
where the `{code}` is a vimL expression.

`fake#gen({keyname})` will return `eval({code})` if the `{keyname}` is defined.
Otherwise, it will search `{keyname}` dictionary on `g:fake_src_paths` and
return a random line of the one.
The later code is equivalent to `fake#choice(fake#load({keyname}))`.


There are some definition examples,

```vim
"" Choose a random element from a list
call fake#define('sex', 'fake#choice(["male", "female"])')

"" Get a name of male or female
"" fake#int(1) returns 0 or 1
call fake#define('name', 'fake#int(1) ? fake#gen("male_name")'
                                  \ . ' : fake#gen("female_name")')

"" Get a full name
call fake#define('fullname', 'fake#gen("name") . " " . fake#gen("surname")')

"" Get a nonsense text like Lorem ipsum
call fake#define('sentense', 'fake#capitalize('
                        \ . 'join(map(range(fake#int(3,15)),"fake#gen(\"nonsense\")"))'
                        \ . ' . fake#chars(1,"..............!?"))')

call fake#define('paragraph', 'join(map(range(fake#int(3,10)),"fake#gen(\"sentense\")"))')

"" Alias
call fake#define('lipsum', 'fake#gen("paragraph")')
```

After these definitions,

```vim
:echo fake#gen("sex")
female

:echo fake#gen("fullname")
Janet Berry

:echo fake#gen("sentense")
Explicari con hendrerit dolore efficiendi laoreet!

:echo fake#gen("paragraph")
Vim vitae fugit vel torquatos! Neque veniam summo tamquam nulla
hendrerit mucius at commodo repudiare sint sodales in. Audiam
detracto erant enim laudem error error lucilius definitiones integre.
Eam veri gravida recteque sit con te agam posidonium fabulas
omittantur diam feugiat noster. Denique bibendum exerci populo usu
netus exerci. Fusce mutat posidonium el est nominati o iriure nec
quidam soluta accusam? Felis pericula leo aeque turpis ne illum integer.
Cetero nullam sonet zril vulputate. Cetero corrumpit quisque nam doming
turpis mutat solet.
```


## Weighted Choice

```
* fake#load({keyname})
	  Search {keyname} dictionary on g:fake_src_paths and
	  return a lines list of the one.

* fake#choice({list})
	  Return a random element from the list

* fake#get({list}, {rate})
	  Return an element at the {rate} from the list.
	  {rate} must be a floating point number between 0.0 and 1.0
	  This is equivalent to list[floor(len(list) * rate)]

* fake#betapdf({a}, {b})
	  Return a random floting point number between 0.0 and 1.0 that
	  has the occurrence of the Beta probability distribution,
	  where the shape parameters a,b > 0
```

The `fake#choice({list})` chooses a random element of the given list,
which its choice is uniformly flat and not biased.
If you need weighted choice, try a combination of `fake#get()` and `fake#betapdf()`.

`fake#betapdf()` is useful as the second argument of `fake#get()`, and its shape chart
indicates its occurrence tendency or frequency. Refer to the following websites
about beta distribution or a way to decide the shape parameters _a,b_.

[Beta distribution - Wolfram|Alpha](http://www.wolframalpha.com/input/?i=beta+distribution)  
[http://keisan.casio.com/exec/system/1180573226](http://keisan.casio.com/exec/system/1180573226)

To tell the truth, the built-in "country" dictionary is ordered by population.
If you need to frequently get "China" or "India" which has huge population,

```vim
"" Get a country weighted by population distribution
call fake#define('country', 'fake#get(fake#load("country"),'
                        \ . 'fake#betapdf(0.2, 4.0))')

:echo fake#gen("country")
China
:echo fake#gen("country")
India
:echo fake#gen("country")
China
```

##### The chart of betapdf(0.2, 4.0)
![betapdf(0.2, 4.0)](https://qiita-image-store.s3.amazonaws.com/0/67544/911d3e93-b76f-cf41-de7b-e55dac920ffe.png "The chart of betapdf(0.2, 4.0)")

##### Other examples

```vim
"" Get an age weighted by generation distribution
call fake#define('age', 'float2nr(floor(110 * fake#betapdf(1.0, 1.45)))')

"" Get a domain (ordered by number of websites)
call fake#define('gtld', 'fake#get(fake#load("gtld"),'
                        \ . 'fake#betapdf(0.2, 3.0))')

call fake#define('email', 'tolower(substitute(printf("%s@%s.%s",'
                        \ . 'fake#gen("name"),'
                        \ . 'fake#gen("surname"),'
                        \ . 'fake#gen("gtld")), "\\s", "-", "g"))')
```


## Add More Dictionaries

You can add your favorite dictionaries.
`g:fake_src_paths` is a list of directory that contains dictionaries,
which will be used in `fake#load()` or `fake#gen()` as its search path.

```vim
let g:fake_src_paths = ['/home/user/.vim/fake_src1',
                        \ '/home/user/.vim/fake_src2']
```


## Others

Please note the following two points,

* This plugin uses `sha256()` as a pseudo-random generator, so that means its
  distribution uniformity maybe worse than the other generator like
  Mersenne Twister or XorShift.

* The built-in dictionaries are subject to change in future for
  quality improvement without any announcement. If you have dissatisfaction
  with the contents, please add your original dictionary on the `g:fake_src_paths`.
