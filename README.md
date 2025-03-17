# Teste Programador PHP LogManager - Suporte

Este teste tem como objetivo conhecer um pouco mais como você programa, e como você faz uma interação com um atendimento de Suporte.

Vamos dividir o teste em 2 etapas.

1 - Análise do seu conhecimento em Desenvolvimento

2 - Análise do seu atendimento em um Chamado



## 1. Análise do seu conhecimento em Desenvolvimento


### O que você precisa fazer

Você precisa criar um mecanismo para sincronizar contas do mercado livre e ressincronziar o token que expira de 6 em 6 horas.

Documentação: https://developers.mercadolivre.com.br/pt_br/autenticacao-e-autorizacao

Para isto, você precisa criar um app de teste: https://developers.mercadolivre.com.br/devcenter/create-app

Pode utilizar o ambiente de homologação do mercado livre mesmo, não é necessário publicar o app em produção.

Também precisa criar um front end e back end desacoplados para criação de um produto simples.

Desafio proposto:
- 1 - Criar um app no mercado livre
    - 1.1 - Sincronizar contas no seu app e recuperar o access token ( e salvar no banco )
    - 1.2 - Ressincronizar quando o token expirar, o token expira a cada 6h
    - 1.3 - Documentação: https://developers.mercadolivre.com.br/pt_br/autenticacao-e-autorizacao
- 2 - Back end e Front end para publicação de produto (não são necessários todos atributos, apenas: titulo, categoria, preço e estoque)
 - 2.1 - Documentação: https://developers.mercadolivre.com.br/pt_br/publicacao-de-produtos
  

Obs 1: de preferência em utilizar o banco de dados MySQL para que seja fácil nossa analise, e também por que é o que utilizamos aqui, ou sqlite.

Obs 2: não esqueça de criar um README.md explicando oque precisamos fazer para rodar sua aplicação aqui em nossa máquina.

Obs 3: não se preocupe em fazer um front-end bonito, mas ele precisa estar desacoplado do projeto e ter uma conexão com a API.


## 2. Análise do seu atendimento em um Chamado

### Contexto

A LogManager, empresa especializada em entregas no mesmo dia, enfrenta desafios críticos no monitoramento de suas entregas. O time de operações identificou problemas que precisam ser analisados e corrigidos por você, o desenvolvedor responsável pelo chamado.

#### Chamado #742

Falha na Atualização do Status das Entregas

- Entregas não estão sendo atualizadas para o status "ENTREGUE", mesmo quando o motorista confirma a entrega.
- Entregas voltando para o status "EM ROTA" automaticamente (sem que nenhuma atualização seja feita).
- Há registros de pacotes sendo entregues antes de serem despachados, indicando um erro na ordem dos eventos.

Os problemas parecem ocorrer em situações onde há múltiplos eventos registrados para uma mesma entrega em um curto período de tempo.

Consulta Extrema no Relatório de Entregas

- O relatório diário de entregas demora mais de 30 segundos para ser gerado, travando o painel de controle.
- As consultas se tornam ainda mais lentas quando há filtros por múltiplos status e transportadoras ao mesmo tempo.

Você deve investigar e corrigir esses problemas.

### O que Você Deve Fazer

Correção do Código:

O código abaixo contém falhas que causam os problemas relatados.

- Identifique e corrija os erros.
- Explique cada problema encontrado e a solução adotada.
- Envie o código atualizado.


Otimização da Query de Relatório de Entregas:

- O código contém uma query base que precisa ser otimizada.
- A otimização deve melhorar o desempenho para grandes volumes de dados e lidar com filtros complexos.
- Explique as melhorias aplicadas

Resposta para o Suporte:

- Redigir uma resposta clara e objetiva para o suporte, explicando os problemas encontrados e as correções realizadas abstraindo os conceitos para pessoas que não são desenvolvedoras.

### Código com Erros

```
namespace App\Http\Controllers;
use Illuminate\Http\Request;
use App\Models\Delivery;
use App\Models\Package;
use App\Models\DeliveryEvent;
use Illuminate\Support\Facades\DB;
class DeliveryController extends Controller
{
    public function updateDeliveryStatus(Request $request)
    {
        DB::beginTransaction();
        try {
            $delivery = Delivery::where('tracking_code', $request->tracking_code)->first();
            if (!$delivery) {
                return response()->json(['error' => 'Entrega não encontrada'], 404);
            }
            $delivery->status = $request->status;
            $delivery->save();

            // Insere um novo evento de entrega
            DeliveryEvent::create([
                'delivery_id' => $delivery->id,
                'status' => $request->status,
                'timestamp' => now(),
            ]);
            DB::commit();
            return response()->json(['message' => 'Status atualizado com sucesso']);
        } catch (\Exception $e) {
            DB::rollback();
            return response()->json(['error' => 'Erro ao atualizar status'], 500);
        }
    }
    public function getDeliveryReport(Request $request)
    {
        $query = "SELECT * FROM deliveries d
                  JOIN packages p ON d.id = p.delivery_id
                  WHERE d.status IN ({$request->statuses}) 
                  AND d.courier IN ({$request->couriers}) 
                  AND d.delivery_date BETWEEN '{$request->start_date}' AND '{$request->end_date}'";
        $deliveries = \DB::select($query);
        return response()->json($deliveries);
    }
}
```


### Tarefa Extra: Otimização da Query

A query abaixo é executada diariamente no relatório de entregas, mas está muito lenta.

```
SELECT * FROM deliveries d
JOIN packages p ON d.id = p.delivery_id
WHERE d.status IN ('ENTREGUE', 'FALHA')
AND d.courier IN ('Transportadora X', 'Transportadora Y')
AND d.delivery_date BETWEEN '2024-03-01' AND '2024-03-05'
```
Requisitos da Otimização:

- Melhorar a performance da query sem alterar sua funcionalidade. Quais aspectos você avaliaria para identificar as causas da lentidão?




## No mais...

Ao finalizar o teste, subir em um reposítorio do github privado e compartilhar com o user grazianilog do github e encaminhar um e-mail para graziani@logmanager.com.br com o link do repositório.

---

Boa sorte! Esperamos ter você aqui com a gente :)
