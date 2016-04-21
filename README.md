# Что нам понадобится

1. СMake - cистема сборки C/C++ проектов.
Проверьте, что в консоли доступна команда ``cmake``
(ссылка на дистрибутив - https://cmake.org/download/ )

2. Распакованная библиотека Boost для Visual Studio (в примерах - ``d:\boost``)
Для Boost Graph достаточно заголовочных файлов, компилировать .lib/.dll не требуется.
https://sourceforge.net/projects/boost/files/boost/1.59.0/
:exclamation::exclamation:```осторожно, в boost 1.60 случайно сломана компиляция adjacency_matrix```:exclamation::exclamation:


3. Пакет GraphViz для генерации изображений графов (распаковать архив в любую папку, в примеах - ``d:\users\graphviz``)

# Создаём проект с помощью cmake.

1. CMake умеет создавать проекты C/C++, совместимые с разными системами сборки (Unix Makefile, Visual Studio и др.). Установленный cmake требуется для сборки этих проектов.
2. Создайте в новой папке хранилище git (``git init``) и файл c именем ``CMakeLists.txt``. Добавьте в него следующие строки:
	```cmake
	project(LabGraph) # название проекта
	cmake_minimum_required(VERSION 2.8.5) # минимальная версия cmake
	
	set(Boost_ADDITIONAL_VERSIONS 1.58;1.59;1.60)
	find_package(Boost REQUIRED) # просим найти библиотеку Boost
	include_directories(${Boost_INCLUDE_DIRS}) # добавляем найденный путь к заголовочным файлам
	add_executable(sample sample.cpp)  # будем компилировать sample.exe из одного .cpp-файла
	#target_link_libraries(sample ${BOOST_LIBRARIES}) # автоматически добавить lib-файлы Boost
	```

3. Создайте файл ``make_project.bat`` следующего содержания
	```cmd
	mkdir build
	cd build
	
	cmake .. -G "Visual Studio 14 2015 Win64" -DBOOST_ROOT=d:/boost
	```
Опция ``-G`` уточняет для какой системы сборки создавать проект, ``-DBOOST_ROOT=...`` -подсказка cmake, где искать boost.
Проект создаётся в отдельной папке build, чтобы не замусоривать основную папку.
4. Создайте заготовку главного файла ``sample.cpp`` следующего содержания:
	```c++
	#define _SCL_SECURE_NO_WARNINGS
	#include <boost/graph/adjacency_list.hpp>
	#include <boost/graph/graphviz.hpp>
	#include <iostream>
	#include <fstream>
	#include <string>
	
	using std::cout;
	using std::endl;
	
	int main()
	{
	 setlocale(LC_ALL, "Russian");
	 return 0;
	}
	```
5. Запустите ``make_project.bat`` и убедитесь в отсустствии ошибок. После этого откройте ``LabGraph.sln`` в папке ``build``, сделайте ``sample`` основным проектом (в контекстном меню проекта) и 
убедитесь, что он запускается.
6. Сохраните файл 
https://raw.githubusercontent.com/github/gitignore/master/VisualStudio.gitignore
под именем ``.gitignore`` рядом с CMakeLists.txt и добавьте к нему строку ``build`` (за этой папкой git не будет следить).
7. Сохраните заготовку проекта ``git commit``, добавив к нему все 4 созданных файла.
(можно git gui или из окна Team Explorer в VisualStudio).

# Осваиваем Boost Graph Library (BGL)

http://www.boost.org/doc/libs/1_60_0/libs/graph/doc/quick_tour.html
http://programmingexamples.net/wiki/Boost/BGL

1. Объявим тип данных для нашего графа:
	```c++
	typedef boost::adjacency_list < 
		boost::vecS, // как хранить вершины - в векторе
		boost::vecS, // как хранить ребра из каждой вершины - в векторе
		boost::undirectedS // неориентированный граф
	> MyGraph;
	```
Библиотека Boost Graph не объектно-ориентированная, а шаблонная (всё сделано через C++ templates). Поэтому у графов, рёбер, вершин нет общих предков (абстрактных классов и др.), а окончательное содержимое классов определяется тем, как их используют в программе. В данном случаем компилятор достраивает код для хранения графа на основе списков смежности, при этом вершины и списки выходящих рёбер хранятся в контейнерах ``std::vector``.
http://www.boost.org/doc/libs/1_60_0/libs/graph/doc/using_adjacency_list.html

2. Заполним простейший граф.
	```c++
		MyGraph g(2); // две вершины, для начала
		boost::write_graphviz(cout, g); // напечатать в консоль в формате graphviz
		boost::add_edge(0, 3, g); // добавить ребро. Появятся ли новые вершины?
		boost::write_graphviz(cout, g);
		auto v = boost::add_vertex(g); // добавить вершину
		cout << "Новая вершина получила номер " << v << endl;
		boost::write_graphviz(std::cout, g);
	```

3. Запустите программу. Обратите внимание на три состояния графа;
     *  формат graphviz достаточно очевиден
     *  вершины для нового ребра созданы автоматически
     *  универсальные функции работы с графам не являются методами объекта, а принимают ссылку на графы  или его итераторы (как ``std::copy, std::sort``).
     * add_edge возвращает дескриптор новой вершины. В данном случае - число.
	~~~
	   Не забудьте сохраниться в git!
	~~~
4. Добавим ещё несколько ребер и обратимся к содержимому на низком уровне:
	```c++
	boost::add_edge(0, 1, g);
	boost::add_edge(1, 3, g);
	boost::add_edge(3, 1, g);
	for (auto e : g.m_edges) {
		//g.m_edges -это std::list!
		cout << "Ребро " << e.m_source << e.m_target << endl;
	}
	for (auto w : g.m_vertices) {
		// g.m_vertices - это вектор (из-за vecS)
		// w.m_out_edges - это тоже вектор (из-за второго vecS)
		cout <<"Вершина степени " << w.m_out_edges.size() << endl;
		auto it = w.m_out_edges.begin(); // итератор вектора 

		if (it!=w.m_out_edges.end())  // если есть хоть одно ребро
			cout << "  Первое ребро ведёт в " << it->get_target() << endl;
	}
	```
        
	~~~
	   Не забудьте сохраниться в git!
	~~~

5. Сохраним и автоматически откроем изображение с помощью GraphViz
	```c++
	std::ofstream f("graph.dot");
	boost::write_graphviz(f, g);
	f.close();
	system("D:/Soft/graphviz/bin/dot.exe graph.dot -Kcirco -Tsvg -o graph.svg");
	system("start graph.svg");
	```
	Не забудьте поменять путь к ``dot.exe`` на актуальный для вашей установки. В браузере должна открыться картинка.
        
	~~~
	   git commit
	~~~	

6. Научимся заменять способ хранения графа. Сменим представление графа на матрицу смежности.
	```c++
	typedef boost::adjacency_matrix <boost::directedS> MyGraph;
	// ориентированный граф, неориентированные тут не поддерживаются
	```
	Возникают ошибки компиляции из-за изменения внутренней структуры класса: ``g.m_edges`` и ``g.m_vertices`` теперь не определены. Вместо них теперь матрица смежности ``g.m_matrix``
	Закомментируйте код печати ребер и вершин. Можно попробовать добавить ребро через матрицу
	```c++
	g.m_matrix[0, 2] = 1;
	```
7. Теперь программа компилируется, но выполняется с ошибками. Проблемы вызывает добавление новых вершин - матрица смежности имеет фиксированный размер.
	- закомментируйте добавление через add_vertex
	- выделите сразу при создании графа достаточное количество вершин
8. Обратимся к вершинам универсальным способом:
	```c++
	 auto vs = boost::vertices(g); 
	 // возвращает ПАРУ итераторов - начало и конец условного списка вершин 
	 MyGraph::vertex_iterator start = vs.first, end = vs.second; // MyGraph::vertex_iterator - тип вершинного итератора
	 for (MyGraph::vertex_iterator it = start; it != end; it++) { // двигаемся от начала к концу
		 cout << "Из вершины "<<*it<<" выходит "<< boost::out_degree(*it, g)<<" ребер"<<endl;
		 // *it - тип дескриптора вершины. Обычно это её номер а для некоторых графов - указатель.
		 //  boost::out_degree сам разберётся, какого типа граф g c помощью шаблонов
	 }
	```
	Вместо ``MyGraph::vertex_iterator`` чуть универсальнее использовать ``boost::graph_traits<MyGraph>::vertex_iterator``.
	Альтернативный способ сделать цикл по вершинам:
	```c++
	boost::graph_traits<MyGraph>::vertex_iterator it, last;
	 for (std::tie(it, last) = boost::vertices(g); it != last; it++) {
	 	 // std::tie - объект позволяющий быстро привоить двev переменным пару значений
		 boost::graph_traits<MyGraph>::vertex_descriptor x = *it; // неизвестный заранее ID вершины. Обычно - число
		 cout << "В вершину "<<x<<" входит "<< boost::in_degree(x, g)<<" ребер"<<endl;
	 }
	```
	
9. Точно также можно перечислить рёбра:
	```c++
	boost::graph_traits<MyGraph>::edge_iterator it2, last2;
	for (std::tie(it2, last2) = boost::edges(g); it2 != last2; it2++) {
		 cout << "Ребро " <<*it2<<", "<< it2->m_source <<" -> "<< it2->m_target<<endl;
	}
	```
	Самый короткий способ - использовать STL (итераторы boost вполне совместимы с ``std::copy``, ``std::find``, ``std::for_each``, ...):
	```c++
	auto es = boost::edges(g);
	std::for_each(es.first, es.second, [&](auto &x) {  // [&] делает доступной переменную g
		 cout << boost::source(x, g)<<","<<boost::target(x,g)<<endl;
	});
	```
	``boost::source(x, g)`` - более универсальный способ узнать, откуда идёт ребро (возвращает дескриптор вершины)
	
10. Упражнение: замените оба auto на настоящие типы  (пару итераторов и дескриптор ребра).

	
11. Можно удобно просматривать соседей вершины и выходящиие из неё рёбра (для обоих типов графов):
	```c++
	auto es0 = boost::out_edges(0,g); 
	 cout << "Из вершины 0 выходят ребра ";
	 std::for_each(es0.first, es0.second, [&](auto &x) {cout << x << " "; });
	 cout << endl;

	 auto avs = boost::adjacent_vertices(0, g);
	 cout << "Из вершины 0 выходят ребра в вершины ";
	 std::for_each(avs.first, avs.second, [&](boost::graph_traits<MyGraph>::vertex_descriptor x) {cout << x << " "; });
	 cout << endl;
	```
12. Переключите класс вновь на ``adjacency_list``. Ошибок компиляции не возникнет.
	* для ориентированных графов ``boost::directedS`` на списках смежности функции ``in_edges`` и ``in_degree`` не доступны, их использование придётся закомментировать.
13. Попробуйте сменить способ списков смежности на списочный:
	```c++
	typedef boost::adjacency_list<
		boost::listS, // хранить ребра из каждой вершины в связном списке
		boost::vecS, // как хранить сами вершины - в векторе
		boost::undirectedS // неориентированный граф
	> MyGraph;
	```
	Теперь рёбра можно удалять без опасения замедлить работу (при удалении рёбер из вектора все последующие сдвигаются).
	
14.	Попробуем удалить вершины и рёбра:
	```c++
	boost::remove_edge(1, 3, g); // для связных списков смежности может быть долго (O(степени вершины))
	 
	boost::remove_edge(*boost::edges(g).first, g); 
	// а через дескриптор ребра быстро. Правда, нужно разбираться с итераторами

	boost::remove_vertex(1,g); // проверьте, перенумеруются ли вершины?
	```
	
15.	Если сделать ```typedef boost::adjacency_list<boost::listS, boost::listS ...``` (хранить вершины не в векторе), получим множество ошибок компиляции, так как доступ по номеру теперь не доступен (только получение дескрипторов через итераторы, тольо хардкор) 
	```c++
	typedef boost::adjacency_list<
		 boost::vecS, // как хранить ребра из каждой вершины - в векторе
		 boost::listS, // как хранить сами вершины - в векторе
		 boost::undirectedS // неориентированный граф
	 > MyGraph2;
	 MyGraph2 g2;
	 auto v0 = boost::add_vertex(g2);
	 auto v1 = boost::add_vertex(g2);
	 boost::add_vertex(g2);
	 auto e0 = boost::add_edge(v0,v1,g2);
	 boost::add_edge(*(--boost::vertices(g2).second), *(boost::vertices(g2).first), g2);
	 // что бы это значило? какие вершины соединяются?
	 
	 //boost::write_graphviz(cout, g2); // сохранить этот граф не так просто
	 
	```
	
	
# Раскрасим и подпишем наши графы 

1. Освоим обобщение ассоциативных массивов PropertyMap в boost. Это набор классов для получения значения по ключу,
которые универсальным образом обращаются к разным контейнерам:
	```c++
	typedef boost::graph_traits<MyGraph>::vertex_descriptor vertex_type;
	auto filledProp = boost::static_property_map<std::string, vertex_type>("filled");
	// эта штука по любому ключу возвращает строку "filled"
	cout <<"boost::static_property_map по любому ключу возвращает одно и то же: "<<
		filledProp[1] << filledProp[763] << endl;

	boost::vector_property_map<std::string> vmap; 
	vmap[0] = "green";
	vmap[3] = "red"; // ключи - номер элемента в запрятанном внутри std::vector
	
	```

2. Такие ассоциативные массивы могут использовать все алгоритмы библиотеки Boost Graph. В частности, с их помощью можно сохранять .dot-файлы с дополнительными атрибутами - цвет, ширина, подписи рёбер и т.д.
Пример: сохраним граф с зелёными вершинами и красными рёбрами:
	```с++
	auto vertColor = boost::static_property_map<std::string, MyGraph::vertex_descriptor>("green");
	auto edgeColor = boost::static_property_map<std::string, MyGraph::edge_descriptor>("red");
	auto nodeId = boost::get(boost::vertex_index, g);
	// map с нумерацией вершин  (дескриптор->номер)
	// если нумерация уже есть (вершины в векторе), то ключи=значениям

	boost::dynamic_properties dp;
	dp.property("color", vertColor);
	dp.property("color", edgeColor); // сам разберётся, какой цвет к чему относится
	dp.property("node_id", nodeId);
	 
	std::ofstream f("graph.dot");
	boost::write_graphviz_dp(cout, g, dp);
	boost::write_graphviz_dp(f, g, dp);
	f.close();
	```
	Просмотрите изображение!
	
3. Упражнение. Добавьте аналогично ``style=filled`` к каждой вершине.
4. Упражнение. Раскрасьте три вершины в красный, жёлтый и зелёный цвета с помощью ``boost::vector_property_map<std::string>``.
5. Удобно бывает внедрить свойства прямо **внутрь** вершин и рёбер, а не хранить их в отдельных PropertyMap-ах (это называется BundledProperties). Для этого нужно создать структуру с нужными полями и упомянуть её в шаблоне графа:
	```c++
	static std::string colors[] = { "red","green","yellow" };
	struct VertInfo {
	public:
		std::string color = colors[(int)(3.0*rand()/RAND_MAX)]; // цвет вершины, случайно определяется при её создании
	};
	
	struct EdgeInfo {
	public:
		int weight = rand() % 100; // вес ребра
	};
	
	typedef boost::adjacency_list<
		boost::listS, // как хранить ребра из каждой вершины - в векторе
		boost::vecS, // как хранить сами вершины - в векторе
		boost::undirectedS, // неориентированный граф
		VertInfo, EdgeInfo
	> MyGraph;
	```
6. Получить доступ к этим данным легко - ``boost::get(&VertInfo::color, g)`` возврашает объект для доступа к color вершин.
Например, сохраним их в файл:
	```c++
	dp.property("color", boost::get(&VertInfo::color, g));
	dp.property("label", boost::get(&EdgeInfo::weight, g)); 
	```
7. Потренируйтесь менять свойства вершин и рёбер разными способами:
	```c++
	 // способ 1 - через PropertyMap, ассоциированный с графом
	 auto allColors = boost::get(&VertInfo::color, g);
	 allColors[1] = "blue";

	 // способ 2 - лёгкий. g[1] - это VertInfo&
	 g[1].color = "brown";

	 // способ 3 - итераторно-шаблонный (g[] сам разбирается на чьи свойства возвращает ссылку)
	 // в квадратных скобках - дескриптор ребра, номеров у них нет
	 g[*(boost::edges(g).first)].weight = 700;
	 g[*(boost::out_edges(1,g).first)].weight = 500;
	```

