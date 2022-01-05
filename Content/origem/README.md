# The Big Bang: Containers, Namespaces, CGroups, Runtime e Kubernetes

## Containers

**Históricamente, havia um problema na carga e transporte de mercadorias em návios comerciais, já que as cargas possuiam diferentes tamanhos, formatos e peso. Isto em grande escala, tornava complexo o processo de gerenciamento e transporte. Com este problema em mãos, foi desenvolvido um "padrão" de armazenamento que, indiferente da carga, o gerênciamento e transporte seriam sempre os mesmos, aumentando assim nossa escala e produtividade.**

**Os containers em computação, não são muito diferentes já que ele é um agrupamento de uma aplicação com todas as suas dependências e que compartilha o Kernel do sistema operacional** do host onde ele está executando, seja ela uma máquina física ou virtual.

O container é nada mais que uma imagem em execução. Esta imagem por sua vez deve ser enxuta, contendo apenas o necessário para rodar a aplicação, sem que ocorra nenhuma alteração, criando-se assim a ideia de imutabilidade. **O Kernel compartilhado, proporciona um melhor desempenho devido ao gerenciamento centralizado de recursos, é possível inclusive fazer uma analogia a ideia de: "É melhor ter um único porteiro para um edifício, já que centralizamos nele todo o gerênciamento de entrada no condomínio, imagine agora ter um porteiro por apartamento..."**

Além da facilidade de empacotar aplicações, o uso de containers permite emular um novo sistema operacional e reutilizar os recursos de hardware de uma forma mais inteligente.

Por fim mas não menos importante, um container tem como característica a portabilidade. Como ele deve ser imutável e todas as dependências imbutidas nele, ele irá rodar em qualquer outro sistema que possua por exemplo, um Docker instalado.

## Namespaces
