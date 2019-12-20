Para acessar a interface gráfica do miniPC
1) Fazer um túnel para o front-end do cluster:
ssh -p 8282 soft5g@ppgca.unisinos.br -L 3390:10.0.200.10:3389
2) Usar um cliente RDP (Remmina, Cord, etc.) e configurar
Server: localhost:3390

Para configurar o serviço Web do free5GC (no miniPC):
1) via browser: localhost:3000
2) via browser: localhost:7100
user: qualquer coisa
pass: qualquercoisa@qualquer.coisa
Precisa estar rodando o script start_stf.sh

Túnel:
ssh <user>@ppgca.unisinos.br -L <porta_local>:<nome_nodo>:<porta_webservice>
Onde <user> é o teu usuário para acessar o front-end do cluster (não o mini_pc); <porta_local> é a porta do teu PC que tu queres redirecionar para o túnel; <nome_nodo> é o nome ou endereço IP que demos para o mini PC; e <porta_webservice> é a porta que está executando o Web Service no mini PC
