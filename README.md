# FIAP-quimiport-tech-challenge

## 1. Entendimento do Domínio

### Contexto de Problema
O Porto de Santos é um dos principais pontos de movimentação do Brasil, lidando com produtos químicos que exigem controle cuidadoso, acompanhamento técnico, documentação adequada e classificação de risco. Atualmente, o registro em empresas de controle é feito de forma manual ou descentralizada. Isso dificulta a consulta das informações, o rastreio do status da carga e a validação de regras de segurança essenciais. 

### Resolução de problema
O **QuimiPort** surge para solucionar esse cenário, oferecendo um sistema centralizado para a gestão inicial dessas cargas, bloqueando ou liberando as operações de acordo com as regras de negócio.

### Usuários Envolvidos
O ecossistema engloba os seguintes atores:
* **Operador portuário:** Atua no registro e na linha de frente da movimentação.
* **Responsável técnico:** Profissional que atesta as cargas.
* **Analista de documentação:** Encarregado de inserir e validar os documentos.
* **Analista de qualidade:** Responsável por avaliações e inspeções na carga.
* **Gestor operacional:** Supervisiona as cargas e o fluxo portuário.
* **Administrador do sistema:** Gerencia os cadastros primários de produtos, usuários e verifica possíveis erros.

### Informações Controladas
Para que o controle seja efetivo, o sistema precisa gerenciar:
* Cadastro de produtos e suas respectivas classes de risco.
* Cadastro de documentação de produtos se obrigatório.
* Documentação obrigatória vinculada a cada carga.
* Dados do responsável técnico pela carga.
* Status de andamento e histórico da carga.
* Cadastro das necessidades da carga (refrigera, frágil, etc...).


### Processos da Operação e Decisões
O fluxo operacional envolve cadastrar os produtos químicos de referência, registrar a entrada das cargas físicas, associá-las aos responsáveis, anexar a documentação (caso obrigatório) e se necessário solicitar inspeções. O sistema atua como o tomador de decisão primário para **liberar ou bloquear** a movimentação de uma carga, validando de forma automatizada se todos os requisitos de segurança e documentais foram cumpridos para seguir até a etapa final.

### Riscos e Restrições
Para mitigar os altos riscos de segurança de produtos químicos o domínio impõe restrições severas:
* Um produto químico não pode ser cadastrado sem classe de risco. Nenhuma carga pode ser registrada sem estar associada a um produto válido e sem um responsável técnico definido.
* A quantidade da carga deve ser sempre maior que zero.
* Uma carga não pode ser liberada para movimentação portuária se estiver com pendências de documentação obrigatória ou se estiver com inspeção em andamento.
* Cargas com status bloqueado ou cancelado não entram em movimentação sob nenhuma hipótese.

## 2. Modelagem com Domain Driven Design (DDD)

Para garantir que a complexidade do ecossistema portuário seja bem traduzida para o código, utilizamos os conceitos de DDD. Esta abordagem permite que as regras de negócio fiquem isoladas e protegidas no núcleo da aplicação.

### Linguagem Ubíqua
Vocabulário compartilhado entre os desenvolvedores e os especialistas de domínio:
* **Produto Químico:** O material de referência cadastrado no sistema (o catálogo). Representa as especificações do produto, e não o contêiner físico.
- **Carga Química:** Lote físico e fechado do produto químico que chega ao porto e precisa ser movimentado.
- **Responsável Técnico:** Profissional legalmente capacitado que atesta a conformidade de uma carga.
- **Documentação:** Conjunto de registros e licenças exigidos para atestar a legalidade e a segurança da carga.
- **Carga Retida:** Status temporário indicando que a carga está aguardando alguma aprovação, seja de teste de qualidade ou liberação de documentação.
- **Inspeção:** Processo de avaliação física ou documental realizado pela equipe de qualidade (analistas).
- **Status da Carga:** O estado atual da carga dentro do fluxo portuário (ex: Registrada, Em Inspeção, Bloqueada, Liberada, Cancelada).
- **Risco da Carga:** Classificação de risco que compõe o material (ex: Explosivos, Gases, Líquidos Inflamáveis, Sólidos Inflamáveis).
- **Necessidade da Carga:** Necessidade especial para transporte e armazenamento (ex: Nenhuma, Refrigeração Contínua, Isolamento Térmico, Armazenamento Estritamente Seco, Acolchoamento/Anti-vibração).
- **Remetente:** Cliente (empresa) de origem que está enviando a carga química para o porto.

### Objetos de Valor
Elementos imutáveis que representam características ou medidas sem identidade própria. Eles agrupam atributos relacionados e encapsulam suas próprias regras de validação para garantir que dados inválidos nunca cheguem às Entidades:

- **QuantidadeMedida:** Representa o volume ou peso físico da carga.
    - _Regra:_ É composto por um `valor` (que deve ser estritamente maior que zero) e uma `unidade` (ex: Litros, Toneladas, KG). Não permite a criação de medidas negativas.
    
- **ClassificacaoRisco:** Representa a categoria de perigo do material de acordo com padrões internacionais.
    - _Regra:_ Deve ser um valor estrito e válido dentro de uma lista padronizada (Enum), como _Líquido Inflamável_ ou _Gás Tóxico_.
    
- **NecessidadeCarga:** Representa as condições rigorosas de transporte e armazenamento exigidas (ex: Refrigeração Contínua, Isolamento Seco, Aterramento Elétrico...).
    
- **CNPJ:** Representa a identificação fiscal de um Remetente.
    - _Regra:_ Não é apenas uma _string_; o objeto valida matematicamente se os dígitos verificadores estão corretos e aplica a formatação padrão (`XX.XXX.XXX/XXXX-XX`).
    
- **Endereco:** Agrupa os dados de localização física do remetente (Logradouro, Número, Cidade, Estado, CEP) em um único bloco conceitual.
    - _Regra:_ Garante que um endereço não seja criado faltando informações cruciais, como o CEP ou o Estado.
    
- **Telefone:** Representa o contato do remetente.
    - _Regra:_ Valida se o número possui o formato correto e o código de área (DDD).

### Entidades
Objetos que possuem identidade única (ID) e cujo ciclo de vida é rastreado pelo sistema.

**1. Produto Químico**
- **Responsabilidade:** Representar o catálogo base de materiais, guardando suas propriedades químicas e de segurança definitivas.
- **Atributos:** `id`, `nome`, `classeRisco`, `status`, `descricao`, `pesoUnidade`.
- **Regras:** Não pode ser cadastrado sem nome ou sem classe de risco. Um produto inativo não pode ser associado a novas cargas registradas.
- **Relacionamentos:** Utilizado como referência principal (catálogo) pela entidade Carga Química.

**2. Carga Química**
- **Responsabilidade:** Rastrear o lote físico do material que chega ao porto, gerenciar a transição de seus status e proteger as regras de movimentação portuária.
- **Atributos:** `id`, `idProduto`, `quantidade`, `necessidadeCarga`, `statusAtual`, `observacoes`, `idResponsavelTecnico`, `idRemetente`, `destino`, `listaDocumentos`, `historicoInspecoes`.
* **Regras:** 
	- Não pode ser registrada sem um produto químico associado.
	- Não pode ser registrada se o produto químico associado estiver inativo.
	- A quantidade informada deve ser maior que zero.
	- Toda carga deve possuir um responsável técnico informado.
	- Uma carga não pode ser liberada sem a documentação obrigatória.
	- Uma carga bloqueada ou cancelada não pode entrar em movimentação.
- **Relacionamentos:** Faz referência direta ao `Produto Químico`, ao `Responsável Técnico` e ao `Remetente`. Atua como raiz do agregado (Aggregate Root) sendo a "dona" das entidades de `Documento da Carga` e `Inspeção`.

**3. Responsável Técnico**
* **Responsabilidade:** Identificar o profissional que responde legalmente pelo status da carga.
* **Atributos:** `id`, `nome`, `tipoTecnico`.
* **Relacionamentos:** Vinculado a uma ou mais Cargas Química.

**4. Documento da Carga**
* **Responsabilidade:** Comprovar o pertencimento legal e de segurança.
* **Atributos:** `id`, `tipoDocumento`, `dataEmissao`, `statusAprovacao`.
* **Relacionamentos:** Pertence exclusivamente a uma Carga Química.

**5. Inspeção**
* **Responsabilidade:** Registrar o histórico de avaliações da equipe de qualidade.
* **Atributos:** `id`, `dataInspecao`, `resultado`, `observacoes`, `idResponsavelTecnico`.
* **Relacionamentos:** Pertence a uma Carga Química.

**6. Remetente (Empresa Cliente)**
- **Responsabilidade:** Representar a empresa de origem que está enviando a carga química para o porto.
- **Atributos:** `id`, `razaoSocial`, `cnpj`, `endereco`, `responsavel`, `telefone`.
- **Regras:** O CNPJ deve ser válido. Um remetente não pode ser excluído do sistema se possuir cargas ativas ou em movimentação no porto.
- **Relacionamentos:** Uma Carga Química é originada por um Remetente.

### Agregados
**Agregado Principal: Carga Química**
- **Responsabilidade:** Atuar como a _Root Entity_ (Raiz do Agregado) e estabelecer a fronteira de consistência transacional do sistema. A Carga Química é a "maestrina" que orquestra todo o ciclo de vida do lote no porto, garantindo que o sistema nunca entre em um estado operacional inválido.
- **Composição do Agregado:** A raiz `Carga Química` engloba suas próprias propriedades e objetos de valor (`quantidade`, `necessidadeCarga`, `statusAtual`), referências externas (`idProduto`, `idResponsavelTecnico`, `idRemetente`) e atua como "dona" das coleções internas de entidades dependentes (`listaDocumentos` e `historicoInspecoes`).

**Por que este agregado foi escolhido e quais regras ele protege?** A Carga Química foi escolhida como agregado principal porque entidades como _Documento da Carga_ e _Inspeção_ não possuem significado ou ciclo de vida independente no ecossistema logístico; elas existem única e exclusivamente em função de uma carga.

Ao centralizar as operações na raiz do agregado, a Carga Química protege as seguintes regras de negócio vitais:

1. **Dependência Estrutural:** Impede a criação de uma carga órfã, exigindo que ela nasça vinculada a um Produto Químico válido (e ativo), a um Remetente e a um Responsável Técnico.
2. **Validação Documental:** Atua como um escudo antes da liberação. O agregado impede a transição de status para "Liberada" se a sua `listaDocumentos` não contiver toda a documentação obrigatória aprovada.
3. **Trava de Segurança:** Trava qualquer movimentação ou transição de status caso o `historicoInspecoes` possua alguma inspeção com status pendente de finalização.
4. **Máquina de Estados Unidirecional:** Orquestra o fluxo de status de forma rígida, garantindo que uma carga classificada como "Bloqueada" ou "Cancelada" jamais volte a fluir indevidamente pelo processo de movimentação portuária.

