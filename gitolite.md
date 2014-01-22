Gitolite: private repositories administration
=============================================

Gitolite viene usato per l'hosting remoto di repositories git.  Uitlizza SSH per autenticare gli utenti attraverso un meccansimo di autenticazione a chiave pubblica. 

#0. Installazione

Se siete interessati all'installazione potete dare un'occhiata qui: 

#1. Prima configurazione
L'installazione prevede la creazione di un utente "gitolite" nel sistema linux (su CentOS l'utente è *gitolite*, se l'installazione è avvenuta manualmente, probabilmente l'avrete chiamato *git*). Su CentOS, la home dell'utente *gitolite* è accessibile al seguente path:

    /var/lib/gitolite

Li troverete tutto quello che vi serve per iniziare.

##1.1 Creazione delle chiavi SSH per l'amministrazione.
La gestione degli utenti e degli accessi ai repository non avviene modificando localmente i file, ma attraverso un repository di amministrazione. Per poter clonare in locale il repository di amministrazione è necessario creare le chiavi RSA per l'utente che amministrerà il sistema in quanto l'accesso ai repository (attraverso i comandi *clone, push, pull, etc.*) è possibile **solo attraverso l'autenticazione a chiave pubblica**. 

**Gitolite non supporta l'autenticazione con password (in realtà non è proprio così, ma la sua configurazione non è semplicissima).**

Per la creazione delle chiavi di amministrazione dei repository, andare sulla propria macchina (quella con cui amministrerete Gitolite) ed eseguite:

    ssh-keygen

Se avete una macchina windows, dovrete installare **msysGit**  (scaricabile da [qui](http://msysgit.github.io/)) e utilizzare **Git Bash**.

Una volta create le chiavi:

    cd ~/.ssh/
    cat id_rsa.pub

Il risultato dovrebbe essere una cosa del genere:
    
    ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAAB [...] redirect@redirect-Aspire-E1-571

Questa è la vostra chiave pubblica: copiatela nella clipboard o salvatela in un file... servirà a Gitolite per confermare la nostra identità.
Ora tornate sulla macchina server dove avete installato Gitolite e da root eseguite:

    [root@localhost ~] passwd gitolite
    Inserire nuova password UNIX: (inserite una password per l'utente gitolite)
    Reinserire la nuova password UNIX: (reinserite la password per conferma)
    [root@localhost ~] su gitolite
    [gitolite@localhost /root] cd ~
    [gitolite@localhost ~] gl-setup -q admin.pub

dove *admin.pub* è un file con all'interno la vostra chiave pubblica precedentemente creata. Tenete traccia del nome del file, dato che il nome che darete al file servirà per identificare il vostro utente nel file di configurazione. 

Usate secure copy o un editor di testo per creare il file sul server remoto 

*gl-setup* è un comando presente nel package di gitolite quindi, se lo avete installato a mano, dovrete specificare il path completo di dove l'avete installato. 

##1.2 Clonazione del repository di amministrazione

Una volta effettuato il setup dell'utente amministratore, tornare sulla propra macchina e digitare:

    git clone gitolite@ipaddress:gitolite-admin.git

se tutto è andato bene, git dovrebbe effettuare la copia del repository di amministrazione sulla propria macchina nella directory corrente.

#2. Gestire le autorizzazioni

Dopo la clonazione:

    cd gitolite-admin

nella directory sono presenti due sotto-directory: */conf* e */keydir*:

- Nella cartella */conf* è presente un unico file *gitolite.conf* che è il file di configurazione di gitolite. 


    