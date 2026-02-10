# M300-STE


10-Toolumgebung
3. GitHub Account & Repository
3.1 GitHub Account

https://github.com
 aufrufen

Benutzerkonto erstellen
E-Mail bestätigen
Anmelden

3.2 Repository erstellen
Neues Repository erstellen
Name: m300-STE
Public
Initialize with README
Repository erstellen

4. Git Bash installieren & konfigurieren
4.1 Installation

Git für Windows herunterladen: https://git-scm.com
Installation mit Standardeinstellungen
Git Bash öffnen

4.2 Git konfigurieren
git config --global user.name "USERNAME"
git config --global user.email "EMAIL"

5. SSH-Key für GitHub
5.1 SSH-Key erstellen
ssh-keygen -t ed25519 -C "EMAIL"
Speicherort: Enter
Passwort: optional

5.2 Public Key anzeigen
cat ~/.ssh/id_ed25519.pub

5.3 SSH-Key zu GitHub hinzufügen
GitHub → Profilbild → Settings
SSH and GPG keys
New SSH key
Key einfügen und speichern

6. Repository lokal klonen
cd /c
git clone git@github.com:USERNAME/m300-STE.git
cd m300-STE

7. VirtualBox installieren
Download: https://www.virtualbox.org
Installation mit Standardoptionen
Neustart empfohlen

8. Vagrant installieren
Download: https://www.vagrantup.com
Installation mit Standardoptionen
Test:
vagrant --version

9. Vagrant-Projekt erstellen
cd /c/m300-STE
mkdir vagrant
cd vagrant
vagrant init

10. Apache Webserver automatisiert einrichten
10.1 Vagrantfile konfigurieren
Das Vagrantfile wird so angepasst, dass:
Ubuntu Jammy verwendet wird
Apache automatisch installiert wird
Port 8080 (Host) auf 80 (VM) weitergeleitet wird
/vagrant als Webroot dient

Finales funktionierendes Vagrantfile
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"
  config.vm.boot_timeout = 600

  # Portweiterleitung Host 8080 → VM 80
  config.vm.network "forwarded_port",
    guest: 80,
    host: 8080,
    host_ip: "127.0.0.1",
    auto_correct: true

  # Apache installieren und konfigurieren
  config.vm.provision "shell", inline: <<-SHELL
    set -e
    apt-get update -y
    apt-get install -y apache2

    # /vagrant als Webroot
    sed -i 's#DocumentRoot /var/www/html#DocumentRoot /vagrant#g' \
      /etc/apache2/sites-available/000-default.conf

    # Zugriff erlauben
    if ! grep -q "<Directory /vagrant>" /etc/apache2/apache2.conf; then
      cat >> /etc/apache2/apache2.conf <<EOF

<Directory /vagrant>
  Options Indexes FollowSymLinks
  AllowOverride All
  Require all granted
</Directory>
EOF
    fi

    a2enmod rewrite
    systemctl restart apache2
  SHELL
end

11. VM starten
vagrant up


Status prüfen:

vagrant port


Erwartetes Resultat:

80 (guest) => 8080 (host)
22 (guest) => 2222 (host)

12. Apache testen
12.1 HTML-Datei erstellen (Host)
cd /c/m300-STE/vagrant
echo "<h1>Hallo Apache</h1>" > index.html

12.2 Browser öffnen

http://127.0.0.1:8080

Der Text „Hallo Apache“ wird angezeigt.

13. VM beenden / löschen
vagrant destroy -f
