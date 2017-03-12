---
date: 2010-05-10T00:27:16Z
tags: []
title: Arp spoofing
tumblr_url: http://blog.bigfun.eu/post/585101318/arp-spoofing
url: /2010/05/10/arp-spoofing/
---

Ostatnio w sieci znów krąży trojan, który m.in. wykorzystuje mechanizm arp spoofingu. Spotkaliście się z dziwnym linkiem do skryptu js na początku strony, np. v.freefl.info/day.js? No to klops… ale można spróbować z tym skutecznie powalczyć.
Nie usuniemy tego żadnym antywirusem, bo nie siedzi to na naszym komputerze. możemy zablokować wspomniany link adblockiem/noscriptem i innymi tego typu podobnymi, lecz jest to tylko tymczasowe rozwiązanie, bo wspomniany link w każdej chwili może się zmienić. Ale o co właściwie chodzi?
Komputery wykorzystują protokół arp do translacji adresów IP na adresy MAC urządzeń sieciowych. komputer atakujący podszywa się w czasie takiej translacji pod inny komputer ( najczęściej gateway do internetu), przez co wszystkie zapytania idą poprzez komputer atakujący (tzw. man in the middle ). Dokładniejszy scenariusz tego ataku można przeczytać tu.
Co zrobić z tym fantem?
Przede wszystkim zlokalizować źródło problemu, czyli komputer zarażony wirusem, będący naszym “podszywaczem” w sieci. w małej sieci domowej to nie problem, gorsza sprawa jeśli jest to np. duża sieć akademikowa (jak w moim przypadku), a jej admini nie kwapią się z analizą i wykryciem zainfekowanych maszyn. Rozwiązaniem tego problemu jest statyczne przypisanie komputerom i ich adresom MAC przydzielonych im adresów IP. aby sprawdzić, czy w naszej sieci występuje problem, wykorzystujemy polecenie arp. wydajemy komendę:arp -a (zarówno na linuksie, jak i na windows). Jeśli w zwróconej tablicy mamy dwa wpisy o tym samym adresie mac i dwóch różnych adresach IP, to znak, że coś jest nie tak. możemy sami ustawić statyczne linkowanie adresów (jeśli znamy adres MAC właściwego komputera) poleceniem:arp -s IP_ADDR MAC_ADDR Podany adres możemy wyciągać analizując ruch narzędziami tcpdump (windump na windows) . Możemy również posłużyć się narzędziami stworzonymi w tym celu: arpON na linuksa, oraz anti-arpspoof na windows. Niemniej jednak są to tymczasowe rozwiązania, gdyż dopiero wyeliminowanie zarażonego komputera zapewni pełnię bezpieczeństwa.
P.S. Wpis ten powstał ze względu na dokuczliwość problemu i trudność znalezienia rozwiązania w internecie.
