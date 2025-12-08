# Inventorier les appareils



fichier /etc/ansible/hosts
                                    
    [all_linux]
    # DNS
    ORL-DNS-REDIRECTEUR ansible_host=192.168.140.85 ansible_user=etudiant ansible_password=admin #ansible_become_password=admin
    ORL-DNS-AUTORITE ansible_host=192.168.140.95 ansible_user=etudiant ansible_password=admin #ansible_become_password=admin
    ORL-DNS-AUTORITE-SECOND ansible_host=192.168.140.125 ansible_user=etudiant ansible_password=ORL2026LCA #ansible_become_password=admin
    ORL-DNS-REDIRECTEUR-SECOND ansible_host=192.168.140.135 ansible_user=etudiant ansible_password=ORL2026LCA #ansible_become_password=admin

    # WEB
    ORL-SRV-WEB ansible_host=192.168.140.105 ansible_user=etudiant ansible_password=admin #ansible_become_password=admin
    ORL-AUTORITE-CERT ansible_host=192.168.140.115 ansible_user=etudiant ansible_password=admin #ansible_become_password=admin
    ORL-SRV-WORDPRESS ansible_host=192.168.140.145 ansible_user=etudiant ansible_password=admin #ansible_become_password=admin
    ORL-REVERSE_PROXY ansible_host=192.168.140.155 ansible_user=etudiant ansible_password=admin #ansible_become_password=admin
    ORL-REVERSE_PROXY-SECOND ansible_host=192.168.140.215 ansible_user=etudiant ansible_password=admin #ansible_become_password=admin

    # SERVEUR
    
    ORL-SRV-BDD ansible_host=192.168.140.165 ansible_user=etudiant ansible_password=admin #ansible_become_password=admin
    ORL-SRV-DOCKER ansible_host=192.168.140.225 ansible_user=etudiant ansible_password=ORL2026LCA #ansible_become_password=admin
    ORL-SRV-GRAYLOG ansible_host=192.168.140.235 ansible_user=etudiant ansible_password=SxrcouKGru #ansible_become_password=admin
    ORL-SRV-ANSIBLE ansible_host=192.168.140.240 ansible_user=etudiant ansible_password=ORL2026LCA #ansible_become_password=admin







# Equipements reseaux

