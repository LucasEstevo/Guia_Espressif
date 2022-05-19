 # Guia Rápido p/ programação com ESP-IDF

## Controle GPIO

```C
#include “driver/gpio.h” //Para usar as GPIO´s você deve utilizar está lib
```

 Configurando se o pino selecionado é saída ou entrada de dados:
```C
gpio_set_direction(Pinout, In/Out); 
```
* Pinout =  GPIO que será utilizada<br>
* In/Out = Define se a porta será entrada ou saída de dados;<br>
	* In utilize => GPIO_MODE_INPUT <br>
     * Out utilize => GPIO_MODE_OUTPUT<br>

```C
gpio_set_level(Pinout, int = Level); 
```
* Pinout =  GPIO que será utilizada
* int Level = Define se a porta será setada para High ou Low;
	* High => Utilize o número 1
	* Low => utilize o número 0

Exemplo:

```C
#define LED GPIO_NUM_2 //Definindo um nome para GPIO2
#define Sensor GPIO_NUM_3 //Definindo um nome para GPIO3

gpio_set_direction(LED,GPIO_MODE_OUTPUT); //Configura a porta LED como saída
gpio_set_level(LED, 1); //Coloca a porta LED em nível alto

gpio_set_direction(Sensor,GPIO_MODE_INPUT); //Configura a porta sensor como entrada

```

>⚠ Importante usar portas definidas invés de colocar o número da porta na função. Pois se precisar alterar alguma porta no micro durante o projeto é só mexer em uma linha ⚠ 

</br>

___

## Interrupção em GPIO
```C
#include “driver/gpio.h” //Para usar a interrupção em GPIO você deve utilizar está lib
```
A interrupção atualmente é utilizada nos projetos onde se tem pulser. Porém pode ser utilizada sempre que precisar pegar sinais de maneira quase instantânea na GPIO.

Exemplo de configuração da interrupção:

```C
#define Pulser1_A GPIO_NUM_35

typedef struct
{
    bool isr_gpio_level;
} odo_isr_struct_t;

void gpio_config()
{
    gpio_config_t io_conf; 
    io_conf.intr_type = GPIO_PIN_INTR_NEGEDGE; //Configura a borda de sinal utilizada
    io_conf.pin_bit_mask = (1ULL << Pulser1_A);
    io_conf.mode = GPIO_MODE_INPUT;  //Configura como INPUT
    gpio_config(&io_conf);

    gpio_intr_enable(Pulser1_A);// Habilita o módulo de interrupção (ISR) na GPIO

    gpio_set_direction(Pulser1_A, GPIO_MODE_INPUT); // Configura Pulser1_A como INPUT
    gpio_pullup_dis(Pulser1_A);    // Desabilita o pull-up mode
    gpio_pulldown_dis(Pulser1_A);  // Desabilita o pull-down mode

    gpio_set_intr_type(Pulser1_A, GPIO_INTR_NEGEDGE); // Configura a interrupção no pin Pulser1_A, utilizando a borda de descida
    gpio_install_isr_service(intr_alloc_flags);
    gpio_isr_handler_add(Pulser1_A, isr_handler, NULL); //(GPIO,void que será chamado durante as interrupções,NULL)
}
```
Exemplo de void utilizado para ser chamado durante as interrupções:
```C
/*Captação dos Sinais do Pulser*/
void isr_handler(void *arg)
{
    odo_isr_struct_t data;
    static bool last_gpio_level;
    uint32_t pin = (uint32_t)arg;
    bool cur_gpio_level = gpio_get_level(pin); //cur_gpio_level recebe o valor do estado do pin
    if (cur_gpio_level != last_gpio_level) //Verifica se o sinal anterior é diferente do sinal atual, para evitar contagem de pulsos fantasmas
    {
        /*Sempre que receber sinal do Pulser irá cair dentro deste if*/
        data.isr_gpio_level = cur_gpio_level;
    }
    last_gpio_level = data.isr_gpio_level; //Atribui o estado atual 1 ou 0 em last_gpio_level
}
```

</br>

___

## Print Básico
```C
printf(“%s %d %f”, texto,inteiro,decimal);
```
Essa função é utilizada para realizar prints no monitor serial do ESP, para printar textos fixos pode se colocar só a mensagem dentro das apas. Exemplo: printf(“Hello World 2022”);
Agora para printar informações advindas de variáveis é utilizado um caracter especial dentro das aspas e depois o nome da variável fora das aspas, exemplos:
```C
char Texto[10];
printf(“%s”,Texto); //Para variável char
int Inteiro = 0;
printf(“%d”,Inteiro); //Para variável int
float Float = 0.0;
printf(“%f”,Float); //Para variável float
```
Lembrando que para printar muitas váriaveis em uma função, elas devem seguir a ordem que foram impostas dentro das aspas. 

## Prints com ESP_LOG
```C
#include “esp_log.h” //Para usar os prints ESP_LOG você deve utilizar está lib
```

Para definir uma Tag utilize a seguinte função:
```C
static const char *Nome_da_Tag = "Texto atribuído a Tag";
```

As seguintes funções fazem a impressão de mensagens no monitor serial. É melhor usar essas para o projeto versão final, devido serem acopladas a TAG você consegue colocar TAG´s diferentes para saber de onde está vindo o print. Outro detalhe é que cada uma é impressa em cores diferentes o que facilita na hora de encontrar algum erro.

```C
ESP_LOGE( Tag,”Texto”); //Utilizar p/ mensagem de erro (Vermelho)

ESP_LOGI( Tag,”Texto” ); //Utilizar p/ mensagem de informação(Verde/Azul)

ESP_LOGW( Tag,”Texto” ); ///Utilizar p/ mensagem de de warning (Amarela)
```

Exemplo:
```C
static const char *Tag_M = ":MENU::";

int valor = 585;

ESP_LOGE( Tag_M,”Erro_0x4569”); //Mensagem de erro (Vermelho)

ESP_LOGI( Tag_M,”Valor definido = %d”,valor); //Mensagem de informação(Verde/Azul)

ESP_LOGW( Tag_M,”Teste” ); //Mensagem de de warning (Amarela)
```
</br>

___
## FreeRTOS
* ## Task

    ```C
    //Para usar a função de Task do FreeRTOS você deve utilizar estas lib's
    #include "freertos/FreeRTOS.h"
    #include "freertos/FreeRTOSConfig.h"
    #include "freertos/task.h" 
    ```
    Para criação das Task é necessário definir um Handle e um void. Exemplo:

    ```C
    TaskHandle_t xTaskTestHandle;       //Handle 
    void vTaskTest(void *pvParameters); //Void
    ```

    Agora é necessário criar o void definido anteriormente:
    ```C
    void vTaskTest(void *pvParameters)
    {
        while(1)
        {
            //Aqui vem seu código
            vTaskDelay(pdMS_TO_TICKS(10)); 
            /* Dependendo o número de tarefas dentro da task, é necessário colocar este delay para ela não crashar */
        }
    }
    ```

    A criação da Task se da em duas maneiras no microcontrolador ESP32, devido ele possuir dois núcelos se você suar a criação padrão ele alocará a task em algum núcleo aleatório. Porém pode-se usar a criação com núcleo definido assim poderá nivelar o processamento e inibir a sobracarga de um único núcleo.
   
    ```C
    /* Criação de Task em núcleo aleatório*/
    xTaskCreate(vTaskTest, "Task_Test", configMINIMAL_STACK_SIZE + 3072, NULL, 10, &xTaskTestHandle);
    ```
    Ordem dos Argumentos:
    1. Nome do Void definido
    2. Nome da Task (Você pode escolher)
    3. Configuração da memória que a Task pode usar
    4. NULL
    5. Prioridade
    6. TaskHandle que foi definido
    </br></br>
   
     ```C
    /* Criação de Task em núcleo Definido*/
    xTaskCreatePinnedToCore(vTaskTest, "Task_Test", configMINIMAL_STACK_SIZE + 1024, NULL, 1, &xTaskTestHandle, 1);
    ```
    Ordem dos Argumentos:
    1. Nome do Void definido
    2. Nome da Task (Você pode escolher)
    3. Configuração da memória que a Task pode usar
    4. NULL
    5. Prioridade
    6. TaskHandle que foi definido
    7. Núcleo 1 ou 0

    </br>

    **configMINIMAL_STACK_SIZE** => Define o valor mínimo de memória que a Task necessita, porém quanto mais processos forem adicionados dentro dela mais memória ela irá consumir. Caso reporte o erro de Watchdog você deve aumentar essa memória colocando ``configMINIMAL_STACK_SIZE + 1024``, ou valores multiplos de 1024(2048,3072,4096...) até parar o erro.

    >Nota: Não é recomendável utilizar a função printf dentro da Task sendo que muitas vezes ela irá fazer a task crashar.

    **Prioridade** => As task possuem a capacidade de rodar simultaneamente junto as demais partes do código. Seria como rodar dois whiles ao mesmo tempo, devido a isso é necessário definir um nível de prioridade para caso o software precise desacelerar alguma ele irá começar cortando as baixas prioridades. Sendo o nível mais baixo = 1 e os altos são a partir disso. Geralmente é usual níveis de prioridade de 1 a 10.

    Pausa/Play de uma Task:
    ```C
    vTaskSuspend(xTaskTestHandle); //Essa função irá pausar a Task
    vTaskResume(xTaskTestHandle); //Essa função irá dar play novamente na Task
    ```
    >Quando a Task for apenas suspensa ela ainda está ocupando memória, o processo seria igual a um notebook/computador em modo de suspensão. Se quiser liberar a memória você deverá deletar a task, porém ao fazer isso você precisará criar novamente a Task caso deseje reutilizá-la

    Deletando uma Task:
    ```C
    vTaskDelete(xTaskTestHandle);
    ```

* ## Queue
    

</br>

___

## Timer

```C
#include "esp_timer.h" //Para usar a função Timer você deve utilizar está lib
```

Para criação das Task é necessário definir um Handle e um void. Exemplo:
```C
static void timer_test(void *args); 
static esp_timer_handle_t timer_test_handler;
```

Agora você precisará criar as configurações do timer:
```C
static esp_timer_create_args_t timer_test_create_args = 
    {
        .callback = &timer_test,
        .name = "Timer_Test",
        .arg = NULL,
};
```

Feito a criação das configurações é hora de criar o timer utilizando a função:
```C
esp_timer_create(&timer_test_create_args, &timer_test_handler); //(Configurações, Handle)
```
>Está função apenas o cria o timer, ela não o inicia.

Para realizar a inicialização do timer é necessário definir se ele será períodico ou one-shot. Caso seja períodico irá ficar se repentindo até que seja pausado e o one shot é executado apenas uma vez.

Períodico:
```C
esp_timer_start_periodic(timer_test_handler, 50000); //(Handle, Micro-Segundos)
```
One-shot
```C
esp_timer_start_onde(timer_test_handler, 50000);//(Handle, Micro-Segundos)
```

Para pausa o timer periódico é necessário usar a seguinte função:
```C
esp_timer_stop(timer_test_handler); //(Handle)
```

Caso deseje excluir o timer criado para liberar memória use:
```C
esp_timer_delete(timer_test_handler); /(Handle)
```

Se você pausar o timer e quiser que ele volte é só dar um start novamente, mas caso delete ele ai você precisará criá-lo de novo para depois dar start.

Exemplo Timer:

```C
/* Definindo mili-segundos, segundos e minutos para simplificar as contas */
#define MSEC_TO_USEC(msec) (msec * 1000)
#define SEC_TO_USEC(sec) (sec * (1000 * 1000)) 
#define MIN_TO_SEC(min) (min * 60)

#define LOG_TIMER_TIMEOUT SEC_TO_USEC(2) //Timer definido com set de 2 segundos

/* Definindo o void e o handle do timer*/
static void timer_test(void *args); 
static esp_timer_handle_t timer_test_handler; 

/* Configurações iniciais do timer */
static esp_timer_create_args_t timer_test_create_args = 
    {
        .callback = &timer_test,
        .name = "Timer_Test",
        .arg = NULL,
};

/*Void Criado para Auxiliar a criação e inicialização do Timer*/
void init_timer_once()
{
    /*Cria o timer*/
    esp_timer_create(&timer_test_create_args, &timer_test_handler); 

    /* Faz a inicialização do Timer como onde-shot, ou seja, irá executar o timer uma única vez.*/
    esp_timer_start_onde(timer_test_handler, LOG_TIMER_TIMEOUT)

    return;
}

void init_timer_periodic()
{
    /*Cria o timer*/
    esp_timer_create(&timer_test_create_args, &timer_test_handler);

    /* Faz a inicialização do Timer como períodico, ou seja, ficará repetindo até receber um comando para finalizar */
    esp_timer_start_periodic(timer_test_handler, LOG_TIMER_TIMEOUT);

    return;
}

void pause_and_delete_timer()
{
    esp_timer_stop(timer_test_handler);
    esp_timer_delete(timer_test_handler);
    return;
}

/*Quando timer acionar irá reportar neste void*/
static void timer_test(void *args)
{
    /*Código*/
    /*De preferência, utilize o mínimo possível de código dentro desta função, para evitar que ela crie algum delay no controle do timer*/
}
```
</br>

___

## UART
```C
#include "driver/uart.h" //Para usar a UART você deve utilizar está lib
```

Definindo TX e RX:
```C
#define TXD_PIN (GPIO_NUM_10)
#define RXD_PIN (GPIO_NUM_9)
```
Inicialização e configuração da UART
```C
void init_uart(void)
{
    const uart_config_t uart_config = {
        .baud_rate = 9600,                      //Configura o Baudrate
        .data_bits = UART_DATA_7_BITS,          //Configura Data Bits 
        .parity = UART_PARITY_ODD,              //Configura Paridade
        .stop_bits = UART_STOP_BITS_1,          //Configura Stop Bits
        .flow_ctrl = UART_HW_FLOWCTRL_DISABLE,  //Configura controle de fluxo
        .source_clk = UART_SCLK_APB,
    };
    uart_driver_install(UART_NUM_1, RX_BUF_SIZE * 2, 0, 0, NULL, 0);
    uart_param_config(UART_NUM_1, &uart_config);
    uart_set_pin(UART_NUM_1, TXD_PIN, RXD_PIN, UART_PIN_NO_CHANGE, UART_PIN_NO_CHANGE);
}
```
Enviando Dados na UART
```C
void enviar_dados(const char *data)
{
    static const char *logName = "TX_UART";
    const int len = strlen(data); // Coleta o len da string
    const int txBytes = uart_write_bytes(UART_NUM_1, data, len);
    ESP_LOGI(logName, "Escrito %d bytes: '%s'", txBytes, data);
}
```
Leitura de Dados que chegaram na UART
```C
void coletar_dados()
{
    static const char *RX_TASK_TAG = "RX_UART";
    uint8_t *data = (uint8_t *)malloc(RX_BUF_SIZE + 1);
    const int rxBytes = uart_read_bytes(UART_NUM_1, data, RX_BUF_SIZE, 1000 / portTICK_RATE_MS);
    if (rxBytes > 0)
    {
        data[rxBytes] = 0;
        ESP_LOGI(RX_TASK_TAG, "Lido %d bytes: '%s'", rxBytes, data);
        printf("\n%s\n", data);
    }
    free(data); // Libera a memoria que foi alocada
}
```

[Código de Exemplo]()






