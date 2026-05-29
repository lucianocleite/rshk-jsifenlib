# AGENTS.md

Este documento descreve as regras, diretrizes de desenvolvimento e a arquitetura do sistema `rshk-jsifenlib`. O objetivo é orientar agentes (IA ou desenvolvedores) que forem dar manutenção ou adicionar funcionalidades a este projeto.

## 1. Visão Geral do Sistema

O `rshk-jsifenlib` é um SDK/Library desenvolvido em Java para interagir de forma facilitada com o **SIFEN** (Sistema Integrado de Facturación Electrónica Nacional - Paraguai). O projeto tem como foco abstrair a complexidade de chamadas SOAP, assinatura de XML e configuração de certificados.

## 2. Arquitetura do Sistema

A arquitetura está dividida da seguinte forma, refletindo a estrutura de pacotes:

### 2.1. Facade Principal (`com.roshka.sifen.Sifen`)
É a porta de entrada da biblioteca. O usuário do SDK utiliza esta classe para invocar as operações como consulta RUC, envio de documentos e consultas de lotes de maneira simplificada.

### 2.2. Core (`com.roshka.sifen.core`)
Armazena a lógica de domínio do negócio e as representações dos dados do SIFEN.
- **Beans (`com.roshka.sifen.core.beans`):** Representam as entidades principais, como Documento Eletrônico (`DocumentoElectronico`) e Eventos.
- **Campos (`com.roshka.sifen.core.fields`):** Mapeamento 1-para-1 com as tags e grupos de tags descritos no manual técnico do SIFEN (request e response).
- **Tipos (`com.roshka.sifen.core.types`):** Enumerações e classes de tipos usados pela API do SIFEN.
- **Exceções (`com.roshka.sifen.core.exceptions`):** Tratamento de falhas e erros conhecidos do sistema.

### 2.3. Configuração (`com.roshka.sifen.core.SifenConfig`)
A configuração dita o comportamento global:
- Definição do Ambiente (`DEV` ou `PROD`).
- Configurações do certificado PFX para comunicação mútua.
- Credenciais e CSC (Código de Seguridad do Contribuyente).
- Pode ser carregada dinamicamente via `.properties` ou de forma programática.

### 2.4. Implementação Interna (`com.roshka.sifen.internal`)
Esta camada realiza o "trabalho pesado", ocultando detalhes técnicos:
- **Requisições e Respostas (`request` / `response`):** Empacota e desempacota o payload do cliente no envelope SOAP de requisição.
- **Helpers (`helpers`):** Manipulação de conexão HTTP (`HttpHelper`), TLS/Certificados (`SSLContextHelper`), geração do XML via SAAJ (`SoapHelper`) e a assinatura digital baseada em XMLDSig (`SignatureHelper`).

## 3. Regras e Diretrizes de Desenvolvimento

Ao modificar o código desta biblioteca, considere sempre as seguintes regras:

1. **Gestão de Dependências (Maven):**
   - O projeto é gerenciado por Maven. Adições de bibliotecas externas devem ser estritamente justificadas.
   - A biblioteca orginalmente não possuía dependências externas. No entanto, para suportar o ecossistema moderno do Java (11+), pacotes `jakarta.xml.soap`, `jakarta.xml.ws` e suas implementações como `saaj-impl` estão presentes.

2. **Compatibilidade com Java 8:**
   - Todo o código principal foi desenvolvido para ser executável desde o Java 1.8 (`sourceCompatibility` e `targetCompatibility` = 1.8). Não utilize APIs exclusivas do Java 9+ nas classes do pacote `src/main/java`.

3. **Arquitetura Baseada em SOAP:**
   - O SIFEN opera sobre SOAP. As alterações de estrutura devem respeitar os envelhecimentos e cabeçalhos exigidos pela infraestrutura fiscal do Paraguai.
   - Sempre utilize o `SoapHelper` e o `SignatureHelper` para a construção e assinatura da mensagem.

4. **Nomenclatura Padrão do SIFEN:**
   - Nos pacotes `fields` e `types`, preserve os nomes das propriedades ou prefixos conforme a documentação oficial da SET do Paraguai (ex: `TdDatGralOpe`, `TgCamAE`, etc.). Isso facilita a manutenção quando um novo manual técnico é lançado.

5. **Testes Unitários:**
   - Alterações no comportamento de formatação (`FieldFormatUtil`, etc) ou na estruturação XML devem ser cobertas por testes na pasta `src/test`.
   - Utilizamos o `JUnit` e `Hamcrest` para asserções.

6. **Gerenciamento de Erros:**
   - Evite "engolir" exceções com catch vazios.
   - Qualquer exceção de formatação ou de falha de conexão deve ser mapeada em uma `SifenException`.

## 4. Deploy e Publicação
O `pom.xml` está configurado para publicar o SDK no `Maven Central` através do `OSSRH`. Ao trabalhar com releases, certifique-se de que os plugins de Javadoc e código fonte estão funcionais e de que os binários podem ser assinados com GPG.