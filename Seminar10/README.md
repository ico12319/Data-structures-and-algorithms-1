# AVL дървета.

## Целта за днес
| Операции    | build(x)       |  find(x)       |   insert/delete   | min/max       |   prev/next |
| ----------- | -----------    |-----------     |    -----------    | ----------    | ---------   |
| Binary tree | O(nlog(n))     |   O(h)         | O(h)              |  O(h)         |   O(h)      |
| AVL tree    | O(nlog(n))     |   O(log(n))    | O(log(n))         |  O(log(n))    |   O(log(n)) |

## Балансирани дървета
Двоично дърво, което поддържа log(n) височина при динамични операции се нарича **балансирано дърво**.

Има много схеми за балансиране на дървета което влезе и съществуването на различни видове балансирани дървета (Red-Black tree, 2-3 Tree...)

Първата схема за балансиране на дървета е предложена през 1962 година от Andelson-Velsky и Landis от където идва и името на AVL дървото.

```
Баланс фактор на възел k определяме като: bf(k) = h(k->left) - h(k->right). 
```

```
Целта ни е за всеки възел k bf(k) да е в множеството {-1, 0, 1}.
```

Трябва да поддържаме това свойство при всяко добавяне или премахване на възел. Тук на помощ идват ротациите! 

Тяхната реализация както и тестове за тяхната коректност можете да намерите [тук](https://github.com/stoychoX/Data-structures-and-algorithms/blob/main/Seminar09/solutions.cpp#L75).

## Как разбираме, че сме нарушили балансираността на възел?
Ако bf(k) не е в {-1, 0, 1} то тогава сме нарушили балансираността. Трябва ни начин да пресмятаме баланс фактора на всеки възел колкото се може по - бързо. Както в много неща в този курс, за да пресмятаме бързо баланс фактора на възел ще жертваме малко памет и ще записваме височината на поддървото във всеки възел. Структурата би изглеждала така:

```cpp
template<class T>
struct Node {
    T data;
    Node<T>* left;
    Node<T>* right;
    int height;

    Node(const T& d, Node<T>* l, Node<T>* r, int h) : data {d}, left {l}, right {r}, height{h} {}
};
```
Сега, за константно време можем да смятаме баланс фактора:
```cpp
template<class T>
int getHeight(const Node<T>* r) {
    if(r == nullptr)
        return 0;
    return r->height;
}

template<class T>
int getBalanceFactor(const Node<T>* r) {
    if(r == nullptr)
        return 0;

    return getHeight(r->right) - getHeight(r->left);
}
```
Разбира се, тази сметка е вярна тогава и само тогава когато височините на лявото и дясното поддърво са коректни.

## insert
Искаме да напишем операция, която за логаритмично време добавя елемент в дървото и запазва неговата балансираност. 

В началото започваме по стандартния начин - рекурсивно намираме къде да добавим елемента и го добавяме като листо. След това обаче става интересно - какво става ако този елемент разбалансира дървото? Той увеличава височината на (логаритмичен) брой поддървета и е напълно възможно да направи някой баланс фактор на 2 или -2. 

Използваме факта, че рекурсивно сме слезли до някое листо и когато излизаме от рекурсията чрез ротации поправяме балансираността на дървото.

При добавяне на елемент ни се налага да направим **най - много две ротации**

Също така при добавяне на елемент трябва да променяме височината. Разбрахме се, че я пазим, а това, че добавяме елемент я променя. За щастие това може да стане за константно време:

```cpp
template<class T>
void updateHeight(Node<T>*& r) {
    if(r == nullptr)
        return;
    
    r->height = getHeight(r->left) + getHeight(r->right) + 1;
}
```

Алгоритъмът за добавяне би изглеждал по следния начин:

```cpp
template<class T>
void addElement(Node<T>*& root, const T& elem) {
    // Стигнали сме до дъното на рекурсията
    if(root == nullptr) {
        root = new Node<T>(elem, nullptr, nullptr, 1);
        return;
    }
    else if(root->data < elem) {
        addElement(root->right, elem); // добавяме елемента, но работата ни не е приключила

        balanceRight(root);            // balance проверява дали има дисбаланс в дървото което променихме 
                                       // и ако има го коригира!

        updateHeight(root);            // Добавихме възел, все пак трябва да си оправим височината!
    }
    else if(root->data > elem) {       // Аналогично на горния случай.
        addElement(root->left, elem);

        balanceLeft(root);

        updateHeight(root);
    }
}
```
Как можем да оптимизираме този алгоритъм като знаем, че винаги при вмъкване се правят най - много две ротации?

Логиката по добавянето е почти същата като при стандартното двоично наредено дърво, но този път имаме функциии поддържащи баланса на r.

Да разгледаме какви случаи имаме при дисбалансираността. Нега разгледаме случая в който дясното поддърво е дисбалансирано. Нещата за лявото са абсолютно симетрични!

## Имаме три случая за баланс фактора на дясното поддърво.


## bf(root.right) = 1
![](media/avl-disbalance.png)

Тук виждаме, че просто лява ротация около корена ни свърши работа, супер.

## bf(root.right) = 0
![](media/avl-disbalance-0.png)

Тук също просто лява ротация ни свърши работа.

## bf(root.right) = -1
![](media/avl-disbalance-1.png)

Е, какво направихме с тази лява ротация? Само направихме нещата по - лоши. Лявата ротация не ни върши работа тук и трябва на намерим друг начин да се справим.

В предните два случая нещата с лява ротация работиха. Дали има начин да отида в някой от тях? 

<span style="color:red">
<b>Забелязвам, че ако направя дясна ротация около корена на дясното поддърво (с височина k + 1) вече баланс фактора на дясното поддърво ще бъде 1.</b>
</span>


<span style="color:red">
<b>Това обаче е случай 1, който знаем, че с лява ротация работи. 
</b>
</span>

<span style="color:red">
Тоест просто правя дясна ротация около корена на дясното поддърво (за да отида в сл. 1) и после правим лява ротация около корена.
</span>

Нарисувайте си по подобен на примерите начин как ще изглежда това.

Хайде да го напишем на код:

```cpp
template<class T>
void balanceRight(Node<T>*& root) {
    // Това не трябва да се случва!
    assert(root != nullptr);

    int rootBalance = getBalanceFactor(root);
    int rightSubTreeBalance = getBalanceFactor(root->right);

    // Както видяхме в картинката десния дисбаланс е със стойност 2 :)
    if(balance == 2) {
        // Това е гадния случай в който ми трябва още една ротация.
        if(rightSubTreeBalance == -1) {
            rotateRight(root->right);

            // Променям височините на двете поддървета, които са засегнати от ротацията
            updateHeight(root->right->right);
            updateHeight(root->right);
        }

        // Сега вече или съм в случай 1 или в случай 2 и просто си правя лява ротация
        rotateLeft(root);

        // Отново актуализирам височините на възлите засегнати от ротацията
        updateHeight(root->left);
        updateHeight(root);
    }
}
```

Проверката за ляв дисбаланс ще изглежда по симетричен начин.
При левия дисбаланс баланс факторът при който трябва да оправим нещата няма да е 2 а ще е -2. Също така случая, в който ни трябват две ротации няма да е -1 а ще е 1.

## Премахване на елемент
При стандартните дървета имахме следния алгоритъм:
1. Намери елемента. Ако го няма върни false.
2. Ако елемента е листо просто го изтрий
3. Ако елемента има само ляво или само дясно поддърво просто лявото (или дясното) поддърво заемат неговото място а той бива изтрит.
4. В противен случай намери най малкия елемент в дясното му поддърво. Този елемент става корен а ние изтриваме елемента, който ни беше зададено да изтрием.

В този алгоритъм обаче винаги се премахва едно листо. Това значи, че височината на логаритмичен брой поддървета ще се промени, а на нас това не ни харесва.  

Това което трябва да добавим в алгоритъма е балансиране на пътя, по който сме преминали. Също така трябва динамично да променяме височините така, че те да са коректни.

```cpp

template<class T>
bool removeElement(Node<T>*& root, const T& elem) {
    if(root == nullptr)
        return false;

    if(root->data < elem) {
        bool res = removeElement(root->right, elem);

        // Не е възможно да има десен дисбаланс, понеже променяме дясното поддърво
        updateHeight(root);
        balanceLeft(root);
        return res;
    }
    else if(root->data > elem) {
        bool res = removeElement(root->left, elem);
        updateHeight(root);
        balanceRight(root);
        return res;
    }
    
    if(!root->left && !root->right) {
        delete root;
        root = nullptr;
    }
    else if(root->left && !root->right) {
        Node<T>* toDelete = root;
        root = root->left;
        delete toDelete;

        updateHeight(root);
    }
    else if(!root->left && root->right) {
        Node<T>* toDelete = root;
        root = root->right;
        delete toDelete;

        updateHeight(root);
    }
    else {
        // Това е неефикасно.
        root->data = findMinElement(root->right)->data;
        removeElement(root->right, root->data);

        updateHeight(root);
        balanceLeft(root);
    }

    return true;   
}
```
Коментара "това е неефикасно" има за цел да намекне, че тази реализация на алгоритъма за премахване може да бъде подобрена. Проблемът е, че T може да е всякакъв тип, вкючително и доста тежък за копиране тип. Също така слизаме да намерим минималния възел един път и след това слизаме още един път за да го премахнем. Това са две разходки, което може да се подобри. 

Решението може да стане само с разместване на указатели. Причината да не го реализираме така тук е за да може да се концентрирате не толкова върху техническата реализация а върху самия алгоритъм.

[Тук](https://github.com/stoychoX/AVL-vs-SkipList/blob/main/AVL/AVLTree.hpp#L409) можете да погледнете примерен начин възел да се премахне само чрез разместване на указатели. Това не е единствения начин, можете да се опитате да го направите сами и да pull-request-нете кода за да го обсъдим.

## Дърветата и stl
В стандартната библиотека на c++ няма нещо като std::tree\<T>. Въпреки това stl ни предоставя обекти, които в реализацията си използват балансирани дървета.

### std::set
Сет (или множество) е асоциативен контейнер който съдържа сортирана последователност от обекти от тип Т. Търсене, премахване и добавяне имат логаритмична сложност. В стл сетовете се имплементират чрез червено черни дървета, които също като AVL дървото поддържат логаритмична височина (използвайки друга схема за балансиране).

Както в множествата, обектите в сета **са уникални**. За него можете също да си мислите като за дърво, което при insert не допуска повторения на ключове.

Да видим бърз пример:
```cpp
#include<iostream>
#include<set>

int main() {
	std::set<int> s;

	for (size_t i = 0; i < 10; i++)
		s.insert(i);

	// find метода връща итератор към елемента
	std::set<int>::iterator elem = s.find(4);

	// Това извежда 4
	std::cout << *elem << " ";

	// Отиваме на следващия елемент... Но колекцията е сортирана!
	++elem;
	
	// Това извежда 5
	std::cout << *elem;
}
```

Супер, но какво връща find когато елемента го няма? Как изобщо да премахвам елемент? Следващия пример отговаря на тези въпроси.

```cpp
#include<iostream>
#include<set>

int main() {
	std::set<int> s;

	// Вкарваме 10 пъти 5, но елементите нямат повторения!
	for (size_t i = 0; i < 10; i++)
		s.insert(5);

	// Премахваме 5
	s.erase(5);

	std::set<int>::iterator it = s.find(5);

	// Когато елемента го няма в множеството просто връща end() итератора (както е с много други обекти в stl)
	if (it == s.end())
		std::cout << "Element not found!";
}
```

Та обекта го няма - връща end итератора. Лесна работа. Супер сега обаче наредбата не ми харесва. Искам колекцията ми да е сортирана в обратен ред. Сортировката зависеше единствено от компаратор функцията (rли релацията по която сортирам, която в първия пример по подразбиране е <). Дали не мога да накарам моя сет да сортира по >?
Това би станало по следния начин:

```cpp
#include<iostream>
#include<set>

int main() {
    // std::greater<T> е структура с оператор()
    // Дали ще можем ние да си напишем наша структура с operator() 
    // И да я пуснем като comparator? Опитайте!
	std::set<int, std::greater<int>> s;

	for (size_t i = 0; i < 10; i++)
		s.insert(i);

	for (auto x : s)
		std::cout << x << " ";
}
```

Често ни се налага да държим в множество обекти които не са числа а примерно някакви структури или класове. Да разгледаме следния код:

```cpp
#include<set>

struct Entity {
    int x;
    int y;
};

int main() {
    std::set<Entity> s;

    s.insert({1, 1});
}
```
Тази програма ще ни върне грешка. Все пак std::set държи последователността сортирана. Тук по какво да сортира? Компилатора ни се оплаква, че нямаме operator<. Нека си напишем и да видим какво ще се случи:

```cpp
#include<set>

struct Entity {
    int x;
    int y;

    bool operator<(const Entity& e) const {
        return x < e.x;
    }
};

int main() {
    std::set<Entity> s;

    s.insert({1, 1});
}
```
Сега вече всичко работи. Ами ако ни трябват повторения?

## std::multiset
std::multiset е мултимножество. Освен стандартните функции multiset може да ни каже колко обекта от този тип имаме чрез метода count. Тук метода erase премахва **всички** елементи с подадена стойност! Ако искаме да премахнем един елемент използваме метода extract.

 extract връща елемент от тип node handle който държи в себе си стойността на извадения елемент.

```cpp
#include<set>
#include<iostream>

int main() {
	std::multiset<int> s;
	s.insert(10);
	s.insert(10);
	s.insert(10);
	auto k = s.extract(10);

    // 10 2
	std::cout << k.value() << " " << s.count(10);
}
```
## Предшественици и наследници в множество

## std::map

## std::multimap