# analise-temp-parado
Nesse projeto de análise de dados, foi feito o trabalho de descobrir o tempo total em que cada caminhão ficou parado
% Importar os dados da tabela CSV
dados = readtable('18PorcentoAtivaçãoPorTrigger.csv');

% Converter colunas para datetime se necessário
dados.StartOfSampling = datetime(dados.StartOfSampling, 'InputFormat', 'dd/MM/yyyy HH:mm');
dados.EndOfSampling = datetime(dados.EndOfSampling, 'InputFormat', 'dd/MM/yyyy HH:mm');

% Ordenar os dados por chassis e StartOfSampling para garantir ordem cronológica
dados = sortrows(dados, {'chassis', 'StartOfSampling'});

% Inicializar uma tabela para armazenar o tempo total parado por chassis
chassis_unicos = unique(dados.chassis);
tempo_total_parado_por_chassis = table(chassis_unicos, duration(zeros(length(chassis_unicos),1),0,0,0), 'VariableNames', {'Chassis', 'TempoTotalParado'});

% Calcular o tempo parado para cada chassis
for j = 1:length(chassis_unicos)
    chassis_atual = chassis_unicos{j};
    dados_chassis = dados(strcmp(dados.chassis, chassis_atual), :);
    tempo_total_parado = duration(0,0,0);
    
    for i = 1:height(dados_chassis)-1
        if ~isnat(dados_chassis.StartOfSampling(i+1)) && ~isnat(dados_chassis.EndOfSampling(i))
            tempo_parado = dados_chassis.StartOfSampling(i+1) - dados_chassis.EndOfSampling(i);
            if tempo_parado > duration(0,0,0) % Somente considere intervalos positivos
                tempo_total_parado = tempo_total_parado + tempo_parado;
            end
        end
    end
    
    tempo_total_parado_por_chassis.TempoTotalParado(j) = tempo_total_parado;
end

% Exibir o tempo total parado por chassis
disp(tempo_total_parado_por_chassis)
