# 3-gcp-compute-engine

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



 










