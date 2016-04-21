# Что нам понадобится

1. СMake - cистема сборки C/C++ проектов.
Проверьте, что в консоли доступна команда ``cmake``
(ссылка на дистрибутив - https://cmake.org/download/ )

2. Распакованная библиотека Boost для Visual Studio (в примерах - ``d:\boost``)
Для Boost Graph достаточно заголовочных файлов, компилировать .lib/.dll не требуется.
https://sourceforge.net/projects/boost/files/boost/1.60.0/

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
5. Запустите ``make_project.bat`` и убедитесь в отсустствии ошибок. После этого откройте ``LabGraph.sln`` в папке ``build``, сделайте ``sample`` основным проектом и 
убедитесь, что он запускается.
6. Сохраните файл 
https://raw.githubusercontent.com/github/gitignore/master/VisualStudio.gitignore
под именем ``.gitignore`` рядом с CMakeLists.txt и добавьте к нему строку ``build`` (за этой папкой git не будет следить).
7. Сохраните заготовку проекта ``git commit``, добавив к нему все 4 созданных файла.
(можно git gui или из окна Team Explorer в VisualStudio).

# Осваиваем Boost Graph Library (BGL)

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
	boost::add_edge(1, 2, g);
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
	system("D:/Soft/graphviz/bin/dot graph.dot -Kcirco -Tsvg -o graph.svg");
	system("start graph.svg");
	```
6. Добавим свойства вершин, хранящиеся в дополнительных объектах:
	```c++
	// PropertyMap в boost - это классы для получения значения по ключу,
	// которые универсальным образом обращаются к разным контейнерам

	typedef boost::graph_traits<MyGraph>::vertex_descriptor vertex_type;
	auto filledProp = boost::static_property_map<std::string, vertex_type>("filled");
	// эта штука по любому ключу возвращает строку "filled"
	cout <<"boost::static_property_map по любому ключу возвращает одно и то же: "<<
		filledProp[1] << filledProp[763] << endl;

	boost::vector_property_map<std::string> vmap; 
	vmap[0] = "green";
	vmap[3] = "red"; // ключи - номер элемента в запрятанном внутри std::vector
	
	```
