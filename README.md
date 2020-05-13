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

