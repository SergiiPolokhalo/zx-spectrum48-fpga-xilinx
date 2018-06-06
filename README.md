# zx-spectrum48-fpga-xilinx
zx spectrum48 fpga xilinx

внезапно захотелось странного. на форуме видел туториал от Ewgeny7 по сборке спектрума. но у него все было для альтеры и на VHDL, а я сижу на xilinx и verilog.  
для начала я к проекту прицепил ядро Т80, но он осбиралось с диким количеством warning и очень криво, например умный расстановщик выкидывал из проекта сигнал INT. а как собрать верно мне квалификации не хватает. пришлось искать вариант Z80 который бы точно жил под xilinx.
нашел ядро https://github.com/gdevic/A-Z80. прицепил к тестовому проекту, все собралось корректно и даже стало что-то дрыгать по шинам адреса и данных.  
теперь настало время туториала, я вбил счетчики, генераторы синхронизаций. появилась картинка, стало веселее. при телодвижениях на шине данных памяти что-то происходит на картинке.  
в туториале используется внешний чип SRAM, у меня на плате его нет. точнее есть на плате с spartan 3AN, но там макроячеек даже на процессор не хватает. а на плате с spartan 6 стоит SDRAM на 64мб, коей надо еще и рулить уметь.  
но у нас же есть 32 блока по 18к блочной памяти! ура. делаем из нее 64кб оперативки. и что-то даже появляется на экране.  
теперь переходим к ROM. берем для начала тестовый образ. но BRAM чипа уже кончилась. ну ладно, у нас есть выделенная память, ее аж 130к. генерим корку, проект собирается, но ничего не происходит. развертка есть, черный экран.  
хм. а видеопамять вообще корректно выводится? вытаскиваем картинку-заставку из образа тетриса. и пихаем ее по нужному адресу при генерации RAM.  
вместо картинки фигня, бордюр не мигает, хотя явно что-то происходит. картинка "кипит", при ресете процессора стоит стабильно.  
тут вылезла ошибка записи в регистр xFE. тест отмигал бордюром, пошли полоски.  
и тут Зоркий Глаз разглядел на отладке чипу от Cypress из которой делают логические анализаторы. быстро прошил ей EEPORM saleae и прицепил к проекту. но 8 бит мало! там же два полных порта прицеплено. sigrok помог осуществить мечту.  
sigrok с fx2law работает с непрошитым ничем FX2LP чипом. и показывает 16 бит. но на 12мгц максимум. ибо надо слать в два раза больше данных.
итак, лог анализатор работает. и что мы видим? а фигню мы видим. память не успевает выставить данные! как так, BRAM работает на 500мгц же, а тут не успевает!  
стоп, она у нас синхронная, увеличиваем ей частоту с 14мгц до 150. и появляется картинка!  
ура, пошел тест памяти, прошел, показал, что все корректно. прогон в течении нескольких часов глюков не выявил.  
теперь собираем корку выделенной памяти под romBIOS спектрум48.  
и ждем 3 часа пока она проведет роутинг. еще и не работает что-то.  
3 часа на сборку проекта - это не серьезно! решил использовать 64кб блочной памяти сразу с образом romBiOS. заодно сделал заглушку на запись в нижние 16кб. собралось за привычные 10 минут.  
не работает. стал смотреть на лог анализаторе, что происходит по шинам. а там какой-то бред вообще. собрал с тестовой прошивкой, и вижу явно бред на шине данных. а откуда он берется? ага..... это данные для видеоконтроллера.  
делаем буферный регистр, в который будут защелкиваться данные для процессора и не пересекаться с данныи от видеоконтроллера. это все прозрачный доступ.  
и вуаля! тест пошел! и прошел! и запустилась romBIOS48 и показала приветствие.  
цепляем клавиатуру.  
