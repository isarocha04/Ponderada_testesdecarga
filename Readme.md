# Golang TDD com k6 para testes de carga


## Instalação

### Pré-requisitos
- **Golang**: Go instalado no seu ambiente.
- **k6**: Para executar os testes de carga, precisamos da ferramenta k6. Instalar executando:
  
  ```bash
  brew install k6
 
 
Descrição dos Arquivos

	•	main.go: Implementa uma API simples em Go usada como alvo dos testes.
	•	tests/: Contém os arquivos de configuração para os diferentes testes de carga.
 
 
 ## Implementação baseada em testes

### Implementação API X testes

Os testes foram escritos antes da implementação das funcionalidades.Abaixo está um exemplo da rota principal da API e o teste correspondente.
 
 ```bash
 package main

import (
	"net/http"
	"net/http/httptest"
	"testing"
)

// Teste para verificar se a rota principal retorna a mensagem "welcome"
func TestWelcomeRoute(t *testing.T) {
	// Criei uma requisição HTTP para a rota "/"
	req, err := http.NewRequest("GET", "/", nil)
	if err != nil {
		t.Fatal(err)
	}

	// Criei um gravador de resposta HTTP
	rr := httptest.NewRecorder()
	handler := http.HandlerFunc(welcomeHandler)

	// Executei o handler com a requisição simulada
	handler.ServeHTTP(rr, req)

	// Verifiquei se o código de status é 200 OK
	if status := rr.Code; status != http.StatusOK {
		t.Errorf("status code inesperado: obtido %v, esperado %v", status, http.StatusOK)
	}

	// Verifiquei se o corpo da resposta tem a mensagem esperada
	expected := "welcome"
	if rr.Body.String() != expected {
		t.Errorf("corpo inesperado: obtido %v, esperado %v", rr.Body.String(), expected)
	}
}
 
```

Este teste falhou , pois o handler welcomeHandler ainda não existia. A seguir, implementei o handler para passar no teste:

``` bash
func welcomeHandler(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("welcome"))
}

```
Quando o teste ter o resultado esperado, podemos passar para a implementação de novas funcionalidades,lembrando de sempre verificar o comportamento esperado com testes.


##### 1. Smoke test

O smoke test verifica se a API está funcional sob uma carga mínima. Ele simula um único usuário acessando a API.
```bash
import http from 'k6/http';
import { sleep } from 'k6';

export const options = {
  vus: 1,
  duration: '5s',
};

export default function () {
  http.get('http://localhost:3000');
  sleep(1);
}
````

Executar :
```bash
k6 run tests/smoke-test.js
````

##### 2. Load test

O load test simula um aumento gradual de usuários acessando a API ao longo do tempo.
``` bash

import http from 'k6/http';
import { sleep } from 'k6';

export const options = {
  stages: [
    { duration: '10s', target: 100 },
    { duration: '30s', target: 100 },
    { duration: '10s', target: 0 },
  ],
};

export default function () {
  http.get('http://localhost:3000');
  sleep(1);
}
````

Executar :
```bash
k6 run tests/load-test.js
````

##### 3. Stress test

O stress test coloca uma carga significativa, aumentando o número de usuários.
```bahs
import http from 'k6/http';
import { sleep } from 'k6';

export const options = {
  stages: [
    { duration: '10s', target: 200 },
    { duration: '30s', target: 200 },
    { duration: '10s', target: 0 },
  ],
};

export default function () {
  http.get('http://localhost:3000');
  sleep(1);
}
```
Executar :
```bash
k6 run tests/stress-test.js
````

##### 4. Spike test

O spike test verifica como a API se comporta em um pico inesperado de acessos.
```bash
import http from 'k6/http';
import { sleep } from 'k6';

export const options = {
  stages: [
    { duration: '1m', target: 10000 },
    { duration: '30s', target: 0 },
  ],
};

export default function () {
  http.get('http://localhost:3000');
  sleep(1);
}
```
Executar:
```bash
k6 run tests/spike-test.js
````

##### 5. Soak Test

O soak test verifica o comportamento da API quando temos uma carga constante ao longo de um longo período de tempo.
```bash
import http from 'k6/http';
import { sleep } from 'k6';

export const options = {
  stages: [
    { duration: '2h', target: 100000 },
  ],
};

export default function () {
  http.get('http://localhost:3000');
  sleep(1);
}
``
 ````

 Executar:
 ```bash
 k6 run tests/soak-test.js
````
### 6. Breakpoint Test

O breakpoint test identifica em que ponto em a API atinge seu limite de capacidade. Esse teste simula uma carga crescente até que o sistema comece a falhar, ajudando a determinar o ponto exato em que a aplicação ou a infraestrutura não consegue mais lidar com o número de requisições.

## TDD
## Implementação do TDD

Durante o desenvolvimento da API, é nítido foram adotados conceitos e aplicações de TDD, que da o caminho para a implementação do código por meio da criação de testes antes da escrita do codigo em si. 


##### 1. Escrita dos Testes

Para cada nova funcionalidade da API, a primeira coisa foi escrever um teste que definisse o comportamento esperado. Por exemplo, para a rota principal da API, o teste verificava se ela retornava a resposta correta. Esse processo de começar pelos testes permitiu definir o que era necessário implementar antes de iniciar a codificação.

##### 2. Implementação do código para passar nos testes

Com o teste, o código da funcionalidade foi implementado com o objetivo de fazer o teste passar. Se o teste falhasse, seria necessário um ajuste no código até que o comportamento estivesse de acordo com o esperado. O foco era garantir que o código apenas atendesse aos requisitos definidos pelos testes, sem adicionar complexidade desnecessária.

##### 3. Refatoração

Após o código passar no teste, era realizada uma refatoração, se necessário, para melhorar a estrutura, legibilidade ou desempenho do código. Essa etapa de refatoração foi realizada com confiança, sabendo que os testes continuariam verificando se a funcionalidade estava correta após as mudanças.

##### 4. Ampliação da cobertura de testes

Conforme mais funcionalidades eram adicionadas à API, novos testes eram escritos para cada uma delas. O objetivo era cobrir todos os cenários críticos, desde as funcionalidades básicas até possíveis erros ou condições excepcionais. Isso garantiu que, mesmo com a adição de novas funcionalidades, o comportamento da API continuasse previsível e confiável.

##### 6. Vantagens TDD 


- **Qualidade e Confiança no Código**: Como os testes foram criados antes do código, teve uma maior confiança de que o sistema funcionava conforme o esperado.
- **Prevenção de possíveis Bugs**: Bugs potenciais foram detectados e corrigidos antes da implementação completa, o que reduziu o tempo gasto em correções posteriores.
- **Manutenção**: O código foi continuamente refatorado, o que tornou o sistema mais fácil de entender e de se manter.
- **Documentação através dos testes**: Os próprios testes serviram como uma forma de documentação do sistema, descrevendo como cada parte deveria se comportar.

