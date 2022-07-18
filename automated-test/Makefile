

all : AutomatedTest
	export TZ='UTC' && ./AutomatedTest

AutomatedTest : AutomatedTest.cpp ../src/StorageHelperRK.cpp ../src/StorageHelperRK.h libwiringgcc
	gcc AutomatedTest.cpp ../src/StorageHelperRK.cpp  unittestlib/libwiringgcc.a -DUNITTEST -std=c++11 -lc++ -Iunittestlib -I../src -o AutomatedTest

check : AutomatedTest.cpp ../src/StorageHelperRK.cpp ../src/StorageHelperRK.h libwiringgcc
	gcc AutomatedTest.cpp ../src/StorageHelperRK.cpp unittestlib/libwiringgcc.a -g -O0 -std=c++11 -lc++ -Iunittestlib -I ../src -o AutomatedTest && valgrind --leak-check=yes ./AutomatedTest 

libwiringgcc :
	cd unittestlib && make libwiringgcc.a 	
	
.PHONY: libwiringgcc
