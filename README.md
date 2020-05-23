# 3-gcp-compute-engine

( notatki z Chmurowisko )

---

#### Wylistowanie wszystkich projektów w GCP.
> gcloud projects list

#### Weryfikacja jakie domyślne wartości mam ustawione.
> gcloud config list

#### Wyświetlenie listy konfiguracji. Konfiguracje pozwalają na tworzenie profili pomiędzy, którymi można się przełączać z ich specyficznymi ustawieniami jak projekt, czy region.

> gcloud config configurations list

#### Utworzenie własnej konfiguracji

> gcloud config configurations create szkolachmury

#### Ustawienie projektu.

> gcloud config set project gcp-cloud-276415

#### Wyświetlenie wszystkich dostępnych regionów

> gcloud compute regions list

#### Ustawienie domyślnego regionu

> gcloud config set compute/region us-central1

#### Tworzenie wirtualnej maszyny. Pierwszym parametrem jest nazwa VM oraz dwa atrybuty: typ VM oraz strefa.

> gcloud compute instances create vm1a --machine-type=f1-micro --zone=us-central1-a

#### Tworzenie dysku

> gcloud compute disks create vmdisk1a --size=50GB --zone=us-central1-a

#### Weryfikacja czy dysk został utworzony.

> gcloud compute disks list

#### Podpięcie nowo utworzonego dysku do VM.

> gcloud compute instances attach-disk vm1a --disk=vmdisk1a --zone=us-central1-a

#### Sformatowanie dysku podpiętego do maszyny. W tym celu łączę się z VM poprzez SSH. 

#### Sprawdzam ile mam dysków. Widzę też ID dysków np. "sdb" to ten nowy 50 GB.

> sudo lsblk

#### Komenda do formatowania dysku (gdzie sdb to ID mojego dysku)

> sudo mkfs.ext4 -m 0 -F -E lazy_itable_init=0,lazy_journal_init=0,discard /dev/sdb

#### Stworzenie katalogu "disk2"

> sudo mkdir -p /disk2

#### Zmountowanie

> sudo mount -o discard,defaults /dev/sdb /disk2

#### Nadanie uprawnień do zapisu

> sudo chmod a+w /disk2

#### Wejście do folderu

> cd /disk2

#### Tworzę plik i wchodzę do edytora nano dodając tekst.

> touch file.txt
> nano file.txt

#### Przeglądam zawartość pliku.

> cat file.txt

#### Wyjście z konsoli VM (SSH).

> exit

#### Wykonanie zrzutu dysku.

> gcloud compute disks snapshot vmdisk1a --snapshot-names vmdisk1a-snapshot-1 --zone=us-central1-a

#### Weryfikuję utworzone zrzuty dysku.

> gcloud compute snapshots list

#### Mając zrzut dysku tworzę dysk, a następnie VM w innej strefie. Najpier tworzę VM w innej strefie.

> gcloud compute instances create vm1c --machine-type f1-micro --zone=us-central1-c

#### Utworzenie dysku dla VM na podstawie wcześniej utworzonego zrzutu dysku.

> gcloud compute disks create vmdisk1c --source-snapshot=vmdisk1a-snapshot-1 --zone=us-central1-c

#### Teraz utworzony dysk mogę przenieść do nowej VM.

> gcloud compute instances attach-disk vm1c --disk vmdisk1c --zone=us-central1-c

#### Na zakończenie należy pousuwać wszystkie zasoby. W pierwszej kolejności odpinam dyski z VM. 

> gcloud compute instances detach-disk vm1a --disk vmdisk1a --zone=us-central1-a

> gcloud compute instances detach-disk vm1c --disk vmdisk1c --zone=us-central1-c

## Stworzenie obrazu z istniejącej VM (custom image) oraz eksport do Storage.

1. Tworzę VM, która będzie bazą do stworzenia obrazu z tej VM.
2. Następnie tworzę Zasobnik. Pamięć > Przechowywanie danych ( gs://example-112358 ).
3. Przed wpisaniem komendy do utworzenia obrazu w konsoli zatrzymuje VM, aby zachować konsystencję danych.
 
> gcloud compute images create my-image --source-disk=instance-1-wp-vm --source-disk-zone=europe-west1-c

4. Eksport do Storage ( gdzie wpimage.tar.gz to nazwa pliku w Storage ).

> gcloud compute images export --destination-uri gs://example-112358/wpimage.tar.gz --image my-image

## Preemptible instance VM ( czyli możliwe wywłaszczenie lub usunięcie po 24h; do 80% ceny )

Tworzenie za pomocą konsoli:

> gcloud compute instances create vm-1 --preemptible ( dla VM )

> gcloud container cluster create vm-1 --preemptible ( dla Google Kubernetes )

#### Zasymulowanie eventu tak, aby Preemptible VM "vm-1" sie wyłączył:

> gcloud compute instances create vm-1 --preemptible --zone=europe-west4-c ( tworzenie Preemptible VM )

> gcloud compute instances simulate-maintenance-event vm-1 --zone=europe-west4-c

#### Migracja VM z jednej strefy do drugiej ( z wykorzystaniem API ):

> gcloud compute instances move example-instance-1 --zone us-central1-b --destination-zone us-central1-f

> gcloud compute instances move nazwa_instancji --zone europe-west6-a --destination-zone europe-west6-b

#### Migracja VM z jednego regionu do drugiego ( z wykorzystaniem API ):

1. Ustawiam no autodelete na dysku ( dysk będzie odporny na usunięcia )

> gcloud compute instances set-disk-auto-delete wordpress-1-vm --zone us-west3-b --disk wordpress-1-vm --no-auto-delete

2. Tworzenie Snapshot dysku

> gcloud compute disks snapshot wordpress-1-vm --snapshot-names backup-myrootsnapshot --zone us-west3-b

> gcloud compute disks snapshot wordpress-1-vm --snapshot-names myrootsnapshot --zone us-west3-b

3. Tworzę nowy dysk ze Snapshot w nowym regionie

> gcloud compute disks create wordpress-1-vm-v2 --source-snapshot myrootsnapshot --zone europe-west6-b> 

4. Tworzenie instancji VM na podstawie nowo utworzonego dysku w nowym regionie

> gcloud compute instances create wordpress-1-vm-v2 --machine-type n1-standard-4 --zone europe-west6-b --disk name=wordpress-1-vm-v2,boot=yes,mode=rw

5. Usunięcie starych Snapshot oraz VM

> gcloud compute snapshots delete myrootsnapshot

> gcloud compute instances delete wordpress-1-vm --zone us-west3-b

> gcloud compute disks delete wordpress-1-vm --zone us-west3-b

6. Dodaje do Compute Engine dla firewalla:

> gcloud compute instances add-tags wordpress-1-vm-v2 --tags=wordpress-1-deployment

#### Zadanie 3 v1.

Tło: https://cloud.google.com/certification/guides/cloud-architect/casestudy-mountkirkgames-rev2

# 1. Wybierz odpowiednią strategię migracji w oparciu o dostarczony opis.

Lift and shift 

# 2. W punktach napisz jak podszedłbyś do tej migracji, które kroki zrealizowałbyś w pierwszej kolejności.

a. Utworzenie obrazu maszyny.
b. Umieszczenie obrazu w GCP.
c. Dobranie odpowiedniej wielkości maszyny i utworzenie jej z dostarczonego obrazu.
d. Uruchomienie środowiska, testowanie, monitorowanie zachowania środowiska oraz aplikacji.

Uwagi do rozwiązania:

- Aby zapewnić hight availability oraz skalowalność (jak wynika z opisu zadania) należałoby wykorzystać Managed instance group, gdzie mamy możliwość utworzenia maszyn w środowisku Multiple zones zapewniającym HA oraz autoskalowalność. Jednak na ten moment nie możemy tego zapewnić ze względu na 'brak wiedzy'. (Google Compute Engine boosts high availability controls).
- W powyższym rozwiązaniu (Managed instance group + Multiple zones) mając całe rozwiązanie w jednej maszynie (aplikacja + baza) mogą pojawić się problemy z wydajnością oraz konsystencją danych pomiędzy maszynami.
- Dodatkowo przenosząc rozwiązanie do chmury należałoby rozważyć sprawy licencyjne, czy obecnie używane pozwalają nam na migrację do chmury oraz oczywiście aspekty prawne.

# 3. Firma wymaga, abyś przygotował demo końcowej architektury i zaprezentował je podczas umówionego spotkania w siedzibie firmy.

## Utworzenie VM z dostarczonego obrazu:

#### Utworzenie zmiennych
bucketName="images-pm-v2"
bucketLocation="europe-west3"
imageName="mountkirk-games-image"
vmName="mountkirk-games-vm"
vmType="f1-micro"
vmZone="europe-west1-b"

#### Utworzenie bucketa dla obrazu.

> gsutil mb -c STANDARD -l $bucketLocation gs://${bucketName}/

> gsutil mb -c STANDARD -l europe-west3 gs://images-pm-v2

#### Skopiowanie obrazu do własnego bucketa

> gsutil cp gs://mountkirk-games-image/mountkirk-games.vmdk gs://${bucketName}/mountkirk-games.vmdk

> gsutil cp gs://mountkirk-games-image/mountkirk-games.vmdk gs://images-pm-v2/mountkirk-games.vmdk

#### Import obrazu do własnego repozytorium

> gcloud compute images import $imageName --os=debian-9 --source-file=gs://${bucketName}/mountkirk-games.vmdk

> gcloud compute images import mountkirk-games-image --os=debian-9 --source-file=gs://images-pm-v2/mountkirk-games.vmdk

#### Utworzenie VM na podstawie obrazu

> gcloud compute instances create $vmName --machine-type=$vmType --zone=$vmZone --image=$imageName --tags=http-server

> gcloud compute instances create mountkirk-games-vm --machine-type=f1-micro --zone=europe-west1-b --image=mountkirk-games-image --tags=http-server

# Usunięcie zasobów

#### VM

> gcloud compute instances delete $vmName --zone $vmZone

> gcloud compute instances delete mountkirk-games-vm --zone=europe-west1-b

#### Image

> gcloud compute images delete $imageName

> gcloud compute images delete mountkirk-games-image

#### Bucket

> gsutil rm -r gs://${bucketName}/

> gsutil rm -r gs://images-pm-v2


# Zadanie 3 v2.

Tło: https://szkolachmury.pl/google-cloud-platform-droga-architekta/tydzien-3-compute-engine/zadanie-domowe/
Rozwiązanie: https://drive.google.com/drive/u/0/folders/1l_9q4rboyiovsg3qGyphASOp_bp1617P

#### 1. Wybór odpowiedniej strategii migracji. 

Odpowiednią strategią migracji w tym przypadku będzie metoda Lift and shift. Wynika to z wymagania uruchomienia rozwiązania bez potrzeby modyfikacji lub dostosowywania aplikacji. 

#### 2. Plan przeprowadzenia migracji. 

• wykonanie obrazu dysku z systemu on-premises w odpowiednim formacie pozwalającym na upload do GCP 
• upload utworzonego obrazu na storage w chmurze GCP 
• utworzenie Compute Engine w wybranych regionach/strefach z zaimportowanego obrazu dysku 

#### 3. Demo końcowej architektury. 

• w celu sprostania wymaganiom bezawaryjność zostały zastosowane dwa rozwiązania:

1. użycie dysków regionalnych w celu synchronizacji do innej strefy w tym samym regionie – pozwoli to na ciągłość działania w sytuacji awarii strefy na której aktualnie uruchomiony jest Compute Engine.

2. użycie dodatkowych regionów typu Passive, gdzie za pomocą cyklicznych snapshotów w razie awarii całego regionu typu Active będzie można uruchomić kopię podstawowego Compute Engine.

• w celu sprostania wymaganiom skalowalności obraz maszyny źródłowej został kilkukrotnie powielony w różnych regionach/strefach a za dzielenie ruchu w zależności od obciążania odpowiada Load Balancer.

• w celu sprostania wymaganiom płynnego dostępu do rozwiązania przez użytkowników końcowych obraz wzorcowy Compute Engine został powielony w różnych lokalizacjach geograficznych a zadaniem Load Balancera będzie przydzielanie użytkowników do najbliższej im lokacji. 
Wszystkie powyższe rozwiązania są realizowane przy założeniu, że dane będą w trybie tylko do odczytu a użytkownicy końcowi nie będą ich modyfikowali. 




















 










