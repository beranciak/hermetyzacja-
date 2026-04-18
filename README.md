# hermetyzacja-
Hermetyzacja (często nazywana też enkapsulacją) to mechanizm ukrywania wewnętrznego stanu obiektu przed światem zewnętrznym i wymuszania interakcji z tym obiektem wyłącznie przez ściśle określone metody. 

Polega to na hermetyzacji pewnych argumentów w taki sposób że dostęp do nich jest możliwy jedynie przez określone funkcje.

Modyfikatory dostępu (Access Modifiers):

private: dostęp tylko wewnątrz danej klasy (fundament hermetyzacji).

public: dostęp nieograniczony z zewnątrz.

Metody służące do bezpiecznego odczytu (get) i modyfikacji (set) prywatnych pól. Kluczowe, bo pozwalają na walidację (np. setter nie pozwoli ustawić ujemnej ceny produktu) oraz sprawiają, że pole może być tylko do odczytu (brak settera). Logiczne powiązanie danych (pól) z operacjami na nich (metodami) w ramach jednej klasy. Obiekt przestaje być tylko strukturą danych, a staje się autonomiczną jednostką. Zasada "Czarnej Skrzynki" (Black Box) Użytkownik obiektu widzi tylko jego publiczne metody (interfejs), nie widząc skomplikowanej logiki w środku. Pozwala to na zmianę wewnętrznej implementacji (np. zmianę bazy danych na plik tekstowy) bez konieczności poprawiania kodu, który z tego obiektu korzysta.






Przyklad hermetyzacji argumentu saldo w taki sposob ze operacje na nim mogą zostać wykonane poprzez odpowiednie funkcje.


```C++


#include <iostream>


class KontoBankowe {

private:

    double saldo = 0.0;


public:

    // Zmieniamy void na bool, żeby funkcja mogła zaraportować sukces lub porażkę

    bool wplac(double kwota) {

        if (kwota <= 0) {

            std::cout << "BLAD: Kwota wplaty musi byc wieksza niz zero!\n";

            return false; // Odmawiamy wykonania i zwracamy fałsz

        }

        

        saldo += kwota;

        std::cout << "Wplacono: " << kwota << ". Obecne saldo: " << saldo << "\n";

        return true; // Wszystko poszło dobrze

    }


    bool wyplac(double kwota) {

        if (kwota <= 0) {

            std::cout << "BLAD: Kwota wyplaty musi byc wieksza niz zero!\n";

            return false;

        }

        if (kwota > saldo) {

            std::cout << "BLAD: Odrzucono: Brak wystarczajacych srodkow!\n";

            return false;

        }

        

        saldo -= kwota;

        std::cout << "Wyplacono: " << kwota << ". Pozostalo: " << saldo << "\n";

        return true;

    }


    double sprawdzSaldo() const { 

        return saldo;

    }

};
```
PRZYKLAD 2 
```C++

#include <iostream>
#include <string>
#include <algorithm> // dla std::min

class PostacRPG {
private:
    std::string nazwa;
    int maxZdrowie;
    int obecneZdrowie;
    bool czyZyje;

    // Prywatna metoda wewnętrzna. Nikt z zewnątrz nie może nagle odpalić 
    // postac.umrzyj() z pominięciem mechaniki zadawania obrażeń.
    void umrzyj() {
        czyZyje = false;
        obecneZdrowie = 0;
        std::cout << nazwa << " pada na ziemie. Koniec gry!\n";
    }

public:
    // Konstruktor do tworzenia postaci
    PostacRPG(std::string imie, int hp) {
        nazwa = imie;
        maxZdrowie = hp;
        obecneZdrowie = hp;
        czyZyje = true;
    }

    // Publiczna metoda - jedyny sposób, żeby zrobić postaci krzywdę
    void otrzymajObrazenia(int obrazenia) {
        if (!czyZyje) {
            std::cout << nazwa << " juz nie zyje, kopiesz lezacego!\n";
            return;
        }

        if (obrazenia < 0) return; // Zabezpieczenie przed głupotą

        obecneZdrowie -= obrazenia;
        std::cout << nazwa << " obrywa za " << obrazenia << " punktow.\n";

        // Klasa SAMODZIELNIE pilnuje, czy postać powinna zginąć.
        // Programista wywołujący tę metodę w ogóle nie musi o tym pamiętać.
        if (obecneZdrowie <= 0) {
            umrzyj();
        } else {
            std::cout << "Pozostalo HP: " << obecneZdrowie << "/" << maxZdrowie << "\n";
        }
    }

    // Kolejna kontrolowana akcja
    void wypijMiksture(int punkty) {
        if (!czyZyje) {
            std::cout << "Zmarli nie pija mikstur.\n";
            return;
        }

        // Zabezpieczenie: leczenie nie może przekroczyć maxZdrowie
        obecneZdrowie = std::min(obecneZdrowie + punkty, maxZdrowie);
        std::cout << nazwa << " leczy sie. Obecne HP: " << obecneZdrowie << "/" << maxZdrowie << "\n";
    }
};

int main() {
    PostacRPG wiedzmin("Geralt", 100);

    wiedzmin.wypijMiksture(50);      // Nie przekroczy 100 HP
    wiedzmin.otrzymajObrazenia(80);  // HP spada do 20
    wiedzmin.otrzymajObrazenia(50);  // HP spada poniżej 0, odpala się śmierć
    wiedzmin.wypijMiksture(20);      // Gra to odrzuca, bo postać już nie żyje

    return 0;
}
```

