################################################################################
# Debian Live PXE – Guia de Implementação
# Autor: Alex S. M. de Souza
# Projeto: Amsouza SysAdmin
# Site: www.amsouza.lat
################################################################################

################################################################################
# 1. Estrutura do Servidor PXE
################################################################################
# Diretório onde ficam os arquivos do Debian Live usados no boot PXE
/var/www/pxe/debian/live

# Arquivos principais
filesystem.squashfs	# Sistema de arquivos compactado do Debian Live
initrd.img		# Disco inicial carregado na memória durante o boot
vmlinuz			# Kernel Linux que será iniciado pelo PXE
squashfs-root/		# Diretório criado após extrair o sistema squashfs para edição

################################################################################
# 2. Instalar ferramentas necessárias
################################################################################
# Para sistemas RHEL/CentOS/Fedora
dnf install squashfs-tools -y	# Instala ferramentas para manipular arquivos squashfs

# Para sistemas Debian/Ubuntu
apt install squashfs-tools -y	# Instala utilitário para extrair e recriar o filesystem.squashfs

################################################################################
# 3. Extrair o sistema Debian Live
################################################################################
cd /var/www/pxe/debian/live	# Entra no diretório onde está o sistema PXE
unsquashfs filesystem.squashfs	# Extrai o sistema compactado para edição

# Resultado:
squashfs-root			# Contém todo o sistema Debian Live descompactado

################################################################################
# 4. Entrar no sistema para configuração
################################################################################
chroot squashfs-root /bin/bash	# Entra no sistema Debian Live como se estivesse rodando nele

################################################################################
# 5. Configuração de DNS
################################################################################
# Editar arquivo de resolução de nomes
/etc/resolv.conf

nameserver 192.168.100.20	# Servidor DNS principal da rede
search lab.local		# Domínio padrão utilizado para resolver nomes locais

################################################################################
# 6. Configuração de NTP (sincronização de horário)
################################################################################
apt install systemd-timesyncd -y	# Instala o serviço responsável por sincronizar o horário

# Arquivo de configuração
/etc/systemd/timesyncd.conf

[Time]   # Seção responsável pelas configurações de tempo
NTP=192.168.100.20	# Servidor NTP principal da rede (interno)
FallbackNTP=0.debian.pool.ntp.org 1.debian.pool.ntp.org		# Servidores públicos como backup

################################################################################
# 7. Instalação do Zabbix Agent
################################################################################
apt install zabbix-agent -y	# Instala o agente do Zabbix para monitoramento da máquina

# Configuração do agente
/etc/zabbix/zabbix_agentd.conf
Server=192.168.100.30		# IP do servidor Zabbix que pode consultar o agente
ServerActive=192.168.100.30	# IP do servidor Zabbix que recebe dados ativos
Hostname=debian-pxe		# Nome do host que aparecerá no Zabbix

################################################################################
# 8. Instalação do AnyDesk
################################################################################
apt install anydesk -y		# Instala ferramenta de acesso remoto
# O AnyDesk permite acesso remoto ao Debian Live após o boot PXE
# Ideal para administração remota das máquinas iniciadas pela rede

################################################################################
# 9. Criar usuário administrativo
################################################################################
adduser admin		# Cria usuário administrativo
usermod -aG sudo admin	# Permite que o usuário admin execute comandos administrativos
groups admin		# Mostra os grupos que o usuário pertence
# Resultado esperado: admin : admin sudo users

################################################################################
# 10. Definir senha
################################################################################
passwd admin	# Define senha do usuário admin
passwd root	# Define senha do usuário root (opcional)

################################################################################
# 11. Sair do chroot
################################################################################
exit	# Sai do ambiente chroot e volta ao sistema do servidor

################################################################################
# 12. Recriar o sistema squashfs
################################################################################
mv filesystem.squashfs filesystem.squashfs.old	# Backup do sistema original
mksquashfs squashfs-root filesystem.squashfs -comp xz
# Recria o sistema compactado que será utilizado no boot PXE
# A compressão xz reduz o tamanho do sistema

################################################################################
# 13. Configuração iPXE
################################################################################
#!ipxe

set base http://192.168.100.10/debian/live	# Endereço HTTP onde estão os arquivos PXE

kernel ${base}/vmlinuz boot=live components ip=dhcp fetch=${base}/filesystem.squashfs quiet
# boot=live ativa modo live
# components carrega módulos adicionais
# ip=dhcp pega IP automaticamente
# fetch baixa o sistema squashfs via HTTP

initrd ${base}/initrd.img			# Carrega o initrd necessário para inicializar o sistema
boot						# Inicia o sistema Debian Live

################################################################################
# 14. Resultado esperado após o boot PXE
################################################################################
# Sistema Debian Live iniciado com sucesso
# Rede configurada via DHCP
# DNS configurado corretamente
# Sincronização NTP ativa e funcionando
# Zabbix Agent ativo e pronto para monitoramento
# AnyDesk disponível para acesso remoto
# Usuário administrativo criado com privilégios sudo
# Sistema pronto para suporte remoto e monitoramento de máquinas

################################################################################
# Possíveis melhorias
################################################################################
# Autologin do usuário admin
# Inicialização automática do AnyDesk
# Exibição do IP da máquina na tela durante o boot
# Configuração de hostname automático baseado no MAC da máquina

################################################################################
# Licença
################################################################################
# Este guia foi desenvolvido com fins educacionais e de compartilhamento de
# conhecimento na área de administração de sistemas e infraestrutura.
# O conteúdo pode ser utilizado, estudado e adaptado livremente em ambientes
# de laboratório, aprendizado, testes ou suporte técnico.
# O autor não se responsabiliza por eventuais problemas decorrentes do uso das
# informações apresentadas. Utilize este material por sua própria conta e risco.
