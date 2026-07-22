# Model card — previsão de preços da AMZN

## Escopo

O projeto avalia modelos supervisionados e uma rede recorrente na previsão do preço de fechamento da Amazon. Seu objetivo é comparar abordagens de modelagem de séries temporais e demonstrar um pipeline completo de aquisição, preparação, treinamento e avaliação.

**Uso pretendido:** estudo, demonstração técnica e experimentação.

**Uso não recomendado:** tomada automática de decisão financeira, execução de ordens ou estimativa de risco sem validações adicionais.

## Fontes e período dos dados

- **AMZN:** dataset `meharshanali/amazon-stocks-2025`, obtido com KaggleHub.
- **Período da base principal:** 15/05/1997 a 21/02/2025.
- **Variáveis externas:** DXY, VIX, NASDAQ, MSFT, GOOGL, AAPL e taxa de dez anos dos EUA, obtidas com Yahoo Finance.
- **Variável-alvo:** preço de fechamento (`Close`).

As fontes remotas podem ser revisadas por seus provedores. Uma nova execução, portanto, pode não produzir exatamente os mesmos dados do registro histórico preservado no notebook.

## Abordagens avaliadas

### Lasso e Ridge

- alvo construído por deslocamento temporal para `t+1` e `t+90`;
- divisão cronológica de 80% para treino e 20% para teste;
- padronização ajustada no conjunto de treino;
- Lasso com `alpha=0.01`;
- Ridge com `alpha=1.0`.

### LSTM univariada

- utiliza somente o histórico de `Close`;
- dados a partir de 2020;
- janelas de 30 observações para bases diárias e 10 para semanais;
- duas camadas LSTM, dropout, early stopping e redução da taxa de aprendizado;
- divisão cronológica de 80% para treino e 20% para teste.

### SVR

- atributos defasados em três períodos;
- divisão cronológica de 67% para treino e 33% para teste;
- escalonamento Min-Max separado para entradas e alvo;
- kernel linear, `C=1` e `epsilon=0.01`;
- previsão futura recursiva, realimentando o valor previsto de `Close`.

## Métricas

- **MAE:** erro absoluto médio, na unidade do preço.
- **RMSE:** raiz do erro quadrático médio, mais sensível a erros grandes.
- **R²:** proporção da variação explicada no conjunto avaliado.
- **MAPE:** erro percentual absoluto médio, apresentado na seção da LSTM.

## Interpretação dos resultados

Na execução preservada, o SVR obteve o menor RMSE no teste de um passo com a base original. Lasso e Ridge apresentaram resultados próximos entre si no horizonte imediato. Para `t+90`, o erro aumentou e o R² ficou próximo de 0,48 nos melhores modelos lineares, mostrando a perda de previsibilidade no horizonte mais distante.

Os resultados não formam um leaderboard totalmente comparável: SVR, LSTM e regressões utilizam períodos, tamanhos de teste e estratégias de previsão diferentes. As métricas devem ser usadas para analisar cada experimento em seu próprio protocolo.

## Limitações e próximos passos

1. **Validação única:** o notebook usa um único corte temporal. Uma avaliação walk-forward ou backtesting com múltiplas janelas produziria uma estimativa mais robusta.
2. **Horizontes com unidades diferentes:** `t+90` representa 90 observações no treinamento, enquanto a validação externa registrada busca 90 dias corridos. Bases semanais também transformam 90 passos em 90 semanas. Esses resultados não devem ser tratados como equivalentes.
3. **Seleção de atributos:** as dez maiores correlações são calculadas antes da separação treino/teste. Em uma versão para produção, a seleção deve ser ajustada apenas sobre o treino para evitar viés otimista.
4. **Pré-processamento da LSTM:** o escalonamento foi ajustado sobre a série completa na versão registrada. O procedimento ideal ajusta o scaler somente no treino.
5. **Previsão recursiva:** no SVR e na LSTM, erros de um passo podem se acumular ao longo de 90 iterações.
6. **Variáveis exógenas futuras:** na previsão iterativa do SVR, atributos diferentes de `Close` são mantidos constantes, simplificação que limita a interpretação do horizonte longo.
7. **Aleatoriedade:** os pesos iniciais da LSTM e diferenças nas fontes remotas podem alterar resultados entre execuções.
8. **Ausência de custos e estratégia:** não há simulação de custos de transação, slippage, retorno ajustado ao risco ou comparação com uma estratégia ingênua de mercado.

Próximas evoluções recomendadas:

- criar baselines de último valor e média móvel;
- adotar validação walk-forward;
- encapsular seleção de atributos e escalonamento em pipelines ajustados somente no treino;
- alinhar todos os horizontes em pregões;
- comparar previsão de preço com previsão de retorno e direção;
- registrar versões dos dados, sementes e ambiente de execução;
- adicionar testes automatizados para criação de atributos e alinhamento temporal.

## Considerações éticas

O projeto não considera perfil de risco, situação financeira, horizonte ou objetivos de investidores. Nenhuma previsão deve ser interpretada isoladamente como recomendação de compra, venda ou manutenção de ativos.
