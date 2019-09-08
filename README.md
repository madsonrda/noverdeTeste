# Data Engineering Technical Test

## Setup

Para a realização do teste usei um PC com SO Debian 10, processador AMD Ryzen 3 2200G, 16 GB de memória RAM, 120GB SSD, 1 TB SATA.
Além disso, o PC posuia o spark 2.4.4 ([Download spark2.4.4](https://www.apache.org/dyn/closer.lua/spark/spark-2.4.4/spark-2.4.4-bin-hadoop2.7.tgz)) e o Anaconda ([Download Anaconda](https://repo.anaconda.com/archive/Anaconda2-2019.07-Linux-x86_64.sh)) instalados.

Para a execução do pyspark com o jupyter foi configurada as seguintes variáves de ambiente:
```
export PYSPARK_DRIVER_PYTHON=jupyter
export PYSPARK_DRIVER_PYTHON_OPTS='notebook'
```
Desse modo o jupyter foi executado da seguinte forma:

`pyspark --master local[*] --driver-memory 10g  --executor-memory 2g`

Os arquivos de entrada estão localizados na pasta input, parquet de saida está na pasta output. A pasta temp contém arquivos parquet dos dataframes auxiliares gerados na execução do teste.

## Observações

Inicialmente, fiz o deploy de um cluster na plataforma dataproc da google com a seguinte configuração limitada pela conta gratuita:

+ Master Node com 13GB de RAM e 4 cores
+ 2 Slaves Node com 8GB de RAM e 2 cores cada

Contudo, enfrentei algumas dificuldades relacionadas a infraestrutura do cluter, tais como tempo de heartbeat do driver, perda de executores logo após a inicialização do spark context e java.lang.OutOfMemoryError GC Overhead Limit Exceeded. além disso, tive dificuldades para alterar as configurações de memória do driver e dos executores, que sempre permanencias as configurações default da Google.

Após horas tentando contornar os problemas enfretados na Google cloud, decide por mudar de plataforma. Assim passei a utilizar o PC de um amigo no dia seguinte.

Com o PC apesar de rodar o spark local consegui customizar as configuraçoes do spark na inicialização do kernel do jupyter. Também infrentei o problema "java.lang.OutOfMemoryError GC Overhead Limit Exceede" no PC principalmente após operações com joins e groupBY. Tentei várias configurações de driver e executor, mudar o número de particões com o repartition e coalesce, porém não obtive sucesso.

A solução que encontrei foi escrever arquivos parquet das transformações que fiz nos diferentes dataframes e carrega-los em seguida, como uma espécie de checkpoint.

A validação do dataframe de saida está incompleta e também não foi validado o schema do dataframe de saida no hive, uma vez que o PC utilizado só tinha o spark de modo local e instalar um hive iria causar mais atraso.


