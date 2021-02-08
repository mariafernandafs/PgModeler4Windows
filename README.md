# PgModeler4Windows
Procedimento de Compilação e Instalação do software de modelagem PgModeler em ambiente Windows.

* * * *
## Compilando o PgModeler 

PgModeler é um Software para modelagem de dados que utiliza o Banco de Dados PostgreSQL (RDBMS). É muito útil para importar e exportar dados em diversos formatos e para ter um desgin do schema utilizado com uma visão ampla dos relacionamentos. 

PARTE 1 - PREPARANDO AMBIENTE
1) Em todas as plataformas, o primeiro requisito será o Qt framework. 
A modalidade recomentada é a The LTS (Long Term Support) mas qualquer versão estável 
pode ser usada, certifique-se de que não seja uma versão abaixo da 5.9.9 (A versão utilizada neste guia foi 5.15).
	* https://www.qt.io/download
	
2) Depois, será necessário a instalação do PostgreSQL (A versão utilizada neste guia foi a 13).
	* https://www.postgresql.org/download/windows/
	
3) Finalmente, será necessário o download recente da cópia do código fonte do PgModeler (a versão utilizada neste guia foi a 0.9.3).
	*  https://pgmodeler.io/download?source=true
	
4) Por último, será necessário um compilador para C++/Qt. 
Recomando o software msys2 MingGW, é uma plataforma que facilita a instalação de packages necessários e 
suas dependências incluindo o Qt framework e PostgreSQL (pode ser utilizada a versão mais recente).
	* https://www.msys2.org/ 
	
* Primeiro, instale o MSYS2 (64-bit ou 32-bit conforme versão da arquitetura do computador), e clique no programa no menu iniciar.

No console do programa MSYS2 MinGW, digite os comandos abaixo na ordem correta:

```console
	$ pacman -Suy
 	$ pacman -Suy
  	$ pacman -S base-devel mingw-w64-x86_64-make mingw-w64-x86_64-gcc mingw-w64-x86_64-postgresql mingw-w64-x86_64-qt5
```

* O "pacman" é um gerenciador de pacotes cria inicalmente para o Arch Linux e tem como objetivo tornar possível o fácil gerenciamento de pacotes tanto dos repositórios oficiais quanto do Arch User Repository (AUR) repositório comunitário e não oficial. Possui uma importante capacidade de `sincronização` da máquina local com a máquina remoto. 
Obs.: Os pacotes do Pacman são em formato tar compactado.

Entendendo os Parâmentros do comando do item 4:
```console
	-Suy -> combinação dos parâmentros -S, -u e -y
	-S ou --sync -> Sincroniza pacotes. Os pacotes são instalados diretamente dos repositórios remotos, incluindo todas as dependências necessárias para executar os pacotes.No exemplo abaixo, irá baixar e instalar o qt e todos os pacotes dos quais ele depende:
		$ pacman -S qt 
	-u ou --sysupgrade -> Atualiza todos os pacotes desatualizados.
	-y ou --refresh -> Baixe uma nova cópia do banco de dados do pacote mestre do (s) servidor (es) definido (s) em pacman.conf (todo gerenciador de pacotes possui um banco de dados que contém suas fontes de busca).
```

Passe esta opção duas vezes para habilitar downgrades de pacote; neste caso, o pacman selecionará os pacotes de sincronização cujas versões não correspondem às versões locais. Isso pode ser útil quando o usuário muda de um repositório de teste para um estável.

5) É necessário corrigir o valor das variáveis de ambiente "PGSQL_LIB, PGSQL_INC, XML_INC and XML_LIB" para que o compilador possa encontrar o cabeçalho e bibliotecas para libxml2 e libpq. Abra o arquivo "pgmodeler.pri" que se encontra junto com o código fonte em um editor e procure o seguinte trecho:  

```
	windows {
		!defined(PGSQL_LIB, var): PGSQL_LIB = C:/msys64/mingw64/bin/libpq.dll
		!defined(PGSQL_INC, var): PGSQL_INC = C:/msys64/mingw64/include
		!defined(XML_INC, var): XML_INC = C:/msys64/mingw64/include/libxml2
		!defined(XML_LIB, var): XML_LIB = C:/msys64/mingw64/bin/libxml2-2.dll
		...
	}
```

Atualize os valores conforme a localização dos arquivos nos diretórios em seu sistema. Salve o arquivo (pgmodeler.pri.) e proceda com o próximo passo. 
 
* * * * 
PARTE 2 - COMPILANDO O SOFTWARE

Todos os comandos abaixo devem ser executados no console do programa MSYS2 MinGW com privilégios root 
e a partir do diretório onde o código fonte do PgModeler se encontra (descompactado).

7) exportar as variáveis "QT_ROOT" e "INSTALLATION_ROOT" com os caminhos completos da instalação do Qt 
e do diretório onde o PgModeler deverá ser instalado (recomanda-se a criação um diretório em branco com 
o nome PgModeler) respectivamente
```console
	$ mkdir -p /C/PgModeler
	$ export QT_ROOT=/C/Qt
	$ export INSTALLATION_ROOT=/C/PgModeler
```

8) Na sequência, rode os seguintes comandos:
```console
	$ qmake -r CONFIG+=release PREFIX=$INSTALLATION_ROOT pgmodeler.pro
	$ make
	$ make install
	$ cd $INSTALLATION_ROOT
 	$ windeployqt pgmodeler.exe pgmodeler_ui.dll
```

* qmake - (make do qt) é uma ferramenta de "build system", ou seja, auxilia na automatização de scripts 
para compilar o código fonte. Normalmente é usado em projetos que utilizam o framework Qt, porém pode 
ser usado em qualquer código C e C++;  contém várias funcionalidades que automatizam o processo de 
geração de diretivas (Makefiles). Primeiro passo o qmake gera um arquivo de projeto (.pro) inicial. 

* make - utilitário GNU make para montar grupos de programas - gera o arquivo compilado.
Ao iniciar o comando make sem a especificação de um arquivo (opção -f), o utilitário 
faz a seguinte sequência de busca no diretório atual:

	    1 - GNUmakefile
	    2 - makefile
	    3 - Makefile
Apenas o primeiro arquivo encontrado é executado pelo utilitário.

* make install - instala o  programa já compilado.

* windeployqt (Windows deployment tool) - A ferramenta de implantação do Windows windeployqt 
foi projetada para automatizar o processo de criação de uma pasta implantável contendo as dependências 
relacionadas ao Qt (bibliotecas, importações QML, plug-ins e traduções) necessárias para executar o aplicativo a partir dessa pasta. 

* * * *

PARTE 3 - RESOLVENDO DEPENDÊNCIAS
Depois do sucesso da compilação do código fonte e instalação dos binários, é necessário copiar
algumas dependências para a pasta de instalação do PgModeler e rodar alguns comandos para
fazer com que os binários sejam localizados corretamente.

9) Substitua o valor das variáveis "PGSQL_ROOT", "PGMODELER_SOURCE" e "MSYS2_ROOT" 
para os caminhos atualizados (PGSQL_ROOT - caminho completo para a pasta de instalação do PostgreSQL,
PGMODELER_SOURCE - caminho completo para o diretório de código-fonte do pgModeler e MSYS2_ROOT - caminho 
completo para a instalação do MSYS2)
```console
	$ export PGSQL_ROOT=/C/"Program Files"/PostgreSQL/13/bin
	$ export PGMODELER_SOURCE=/C/Users/User/Downloads/pgmodeler-0.9.3/pgmodeler-0.9.3.tar/pgmodeler-0.9.3
	$ export MSYS2_ROOT=/C/Qt/6.0.1
```		

10) Por fim, rode os comandos abaixo para copiar as bibliotecas do mingw para a pasta onde ficará o PgModeler:
```console
	$ cd $MSYS2_ROOT/mingw64/bin/
	$ cp libicuin*.dll libicuuc*.dll libicudt*.dll libpcre2-16-0.dll libharfbuzz-0.dll \
		  libpng16-16.dll libfreetype-6.dll libgraphite2.dll libglib-2.0-0.dll libpcre-1.dll \
		  libbz2-1.dll libssl-1_1-x64.dll libcrypto-1_1-x64.dll libgcc_s_seh-1.dll \
		  libstdc++-6.dll libwinpthread-1.dll zlib1.dll libpq.dll libxml2-2.dll liblzma-5.dll \
		  libiconv-2.dll libintl-8.dll libbrotlidec.dll libbrotlicommon.dll libdouble-conversation.dll \
		  libzstd.dll $INSTALLATION_ROOT	
```

* * * *

PARTE 4 - EXECUTANDO O PGMODELER
Finalmente o PgModeler pode ser iniciado.
Abra a pasta de instalação e dê um dublo clique na aplicação "pgmodeler.exe" e divirta-se.

Enjoy!

=)

https://pgmodeler.io/support/installation
