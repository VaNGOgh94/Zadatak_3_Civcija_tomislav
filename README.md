#include <iostream>
#include <fstream>
#include <ctime>
#include <cstring>
#include <windows.h>
#include <conio.h>
#include <iomanip>
using namespace std;

struct kliStrukt {
  short ai, bi, ci, di;
  char ime [21], prezime [26], spol;
  int datum [3]; 
};
//#include "red_pokazivac.h"
#include "red_polje.h"

redStrukt 
 *glavniRed=InitQ(glavniRed),
 *posiljkaRed=InitQ(posiljkaRed),
 *racunRed=InitQ(racunRed),
 *lutrijaRed=InitQ(lutrijaRed),
 *unionRed=InitQ(unionRed),
 *evoTvRed=InitQ(evoTvRed),
 *brziRed=InitQ(brziRed);
 
int 
 //aiPosiljka=0,
// aiRacun=0,
// aiLutrija=0,
// aiUnion=0,
// aiEvoTv=0,
 biPosiljka=0,
 biRacun=0,
 biLutrija=0,
 biUnion=0,
 biEvoTv=0,
 prvaDva=0;
struct podaci{
  char ime [21], prezime [26], spol;
};
bool brziSim=false;
short brojElemenata, brElemRed;
time_t t =time(0);
tm *DATUM= localtime(&t);

short dajGodine(kliStrukt osoba){
  short godina=DATUM->tm_year+1900,
  mjesec=DATUM->tm_mon+1,
  dan=DATUM->tm_mday;
  if(osoba.datum[1]>mjesec)
    return godina-osoba.datum[2]-1;
  else if(osoba.datum[1]<mjesec)
    return godina-osoba.datum[2];
  else{
    if(osoba.datum[0]>dan)
      return godina-osoba.datum[2]-1;
    else
      return godina-osoba.datum[2];
  }
}
//1a  2a  1b  3a  1c  2b  3b

//1a 1b 1c 2a 3a 
void genNtorku(kliStrukt &klijent){
  klijent.ai=rand()%491+10;
  klijent.bi=rand()%721+80;
  int br=rand();
  if(br%6==0){
    klijent.di=3;
  }
  else
    klijent.di=br%5+1;
  
  do{
    klijent.ci=rand()%5+1;
    if(dajGodine(klijent) < 65 && klijent.ci==4)//mlađi od 65 a mirovljenik
      continue;
    if(klijent.spol=='m'&&klijent.ci==3)//muško a trudan
      continue;//ponovno generiramo
    break;
  } while (1);
}

void obrniRed(redStrukt* red){
  if(IsEmptyQ(red))
    return;
  kliStrukt pom=FrontQ(red);
  DeQueueQ(red);
  obrniRed(red);
  EnQueueQ(pom, red);
}

void sortPrioritet(redStrukt *red, short domet){
  struct elemLista {
    kliStrukt klijent;
    elemLista *sl;
    };
  elemLista* lista=new elemLista;
  lista->sl=NULL;
  elemLista *trazi;
  kliStrukt pomElem;
  if(domet==1){//dodajem prvog na prvo mjesto
      trazi=lista->sl=new elemLista;
      trazi->klijent=FrontQ(red);
      DeQueueQ(red);
      trazi->sl=NULL;
  }
  while(!(IsEmptyQ(red))){
    pomElem=FrontQ(red);
    DeQueueQ(red);
    trazi=lista;
    if(domet==1)//a dodavam sortirano tek nakon prvog elementa
      trazi=trazi->sl;
    //while(trazi->sl && (strcmp(unos.mBrend, trazi->sl->mBrend)==0) && trazi->sl->cijena<unos.cijena)
    
    while(trazi->sl && trazi->sl->klijent.ci <= pomElem.ci) 
      trazi=trazi->sl;
    
    elemLista* cuvaj=trazi->sl;
    trazi=trazi->sl=new elemLista;
    trazi->klijent=pomElem; 
    trazi->sl=cuvaj;
  }
  for(elemLista *pom=lista->sl; pom; pom=pom->sl){//vraćam vezanu listu u red
    //cout<<pom->klijent.ci<<" ";
    EnQueueQ(pom->klijent, red);
  }
}

void ispis(redStrukt *red){
  if(IsEmptyQ(red))
    return;
  kliStrukt pom=FrontQ(red);
  brElemRed++;
  cout<<setw(12)<<left<<pom.ime<<setw(13)<<left<<pom.prezime<<setw(6)<<left<<pom.ai<<setw(6)<<left<<pom.bi<<setw(4)<<left<<pom.ci<<setw(4)<<left<<pom.di<<endl;
  DeQueueQ(red);
  ispis(red);
  EnQueueQ(pom, red);
}

bool obradiKlijente(redStrukt *red, kliStrukt klijent, int &bi){
  if(IsEmptyQ(red))
    return false;
  int ai=klijent.ai;
  while(1){
    if(bi<=ai){//mušterija je obrađena a novi još nije ušao jer je vrijeme ulaska novog (ai) > vrijeme obrade trenutnog (bi)
      cout<<"Izasao je "<<FrontQ(red).ime<<" "<<FrontQ(red).prezime<<" na salter "<<FrontQ(red).di<<endl;
      DeQueueQ(red);
      ai=ai-bi;
      Sleep(35);
      if(IsEmptyQ(red)){
        bi=0;
        return false;
      }
      bi=FrontQ(red).bi;
    }
    else if (ai<bi) {//ako se klijent obrađuje a novi uđe, neka se novog stavlja na njegovo mjesto (pomocu sorta)
      bi-=ai;
      //cout<<"Zasad valja"<<endl;
      //sortPrioritet(red, 1);
      return true;
    }
  }
}

void dodajKlijente( podaci *imeSpol){
  cout<<"Unesi broj klijenata (minimalno 200): ";
  while(1){
    cin>>brojElemenata;
    if(brojElemenata>0)
      break;
    cout<<"Pogresan unos! Unesite ponovno: ";
  }
  cout<<"///POSLOVNICA ZAPOCINJE POSAO ZA 2 SEKUNDE///"<<endl;
  Sleep(2000);
  kliStrukt klijent;
  bool dosaoPrije;
  
  for(int i=0; i<brojElemenata; i++){
    short random=rand()%52; //za ime i spol
    strcpy(klijent.ime, imeSpol[random].ime);
    klijent.spol=imeSpol[random].spol;
    random=rand()%52;//za prezime
    strcpy(klijent.prezime, imeSpol[random].prezime);
    genNtorku(klijent);
    klijent.datum[0]=rand()%30+1;
    if(rand()%3==0)
      klijent.datum[1]=12;
    else
      klijent.datum[1]=rand()%12+1;
    klijent.datum[2]=rand()%53+1943;//1942 - 1995 godišta (max starost = 72, max mladost = 18)
    EnQueueQ(klijent, glavniRed);
    
    dosaoPrije=false;
    if(klijent.di==1){
      if(!IsEmptyQ(posiljkaRed))
        dosaoPrije=obradiKlijente(posiljkaRed, klijent, biPosiljka);
      else biPosiljka=klijent.bi;
      EnQueueQ(klijent, posiljkaRed);
      sortPrioritet(posiljkaRed, 1);
    }
    else if(klijent.di==2){
      if(!IsEmptyQ(racunRed))
        dosaoPrije=obradiKlijente(racunRed, klijent, biRacun);
      else biRacun=klijent.bi;
      EnQueueQ(klijent, racunRed);
      sortPrioritet(racunRed, 1);
    }
    else if(klijent.di==3){
      if(!IsEmptyQ(lutrijaRed)) 
        dosaoPrije=obradiKlijente(lutrijaRed, klijent, biLutrija);
      else biLutrija=klijent.bi;
      EnQueueQ(klijent, lutrijaRed);
      sortPrioritet(lutrijaRed, 1);
    }
    else if(klijent.di==4){
      if(!IsEmptyQ(unionRed))
        dosaoPrije=obradiKlijente(unionRed, klijent, biUnion);
      else biUnion=klijent.bi;
      EnQueueQ(klijent, unionRed);
      sortPrioritet(unionRed, 1);
    }
    else if(klijent.di==5){
      if(!IsEmptyQ(evoTvRed))
        dosaoPrije=obradiKlijente(evoTvRed, klijent, biEvoTv);
      else biEvoTv=klijent.bi;
      EnQueueQ(klijent, evoTvRed);
      sortPrioritet(evoTvRed, 1);
    }
    cout<<"Usao je "<<" "<<klijent.ime<<" "<<klijent.prezime<<" na salter"<<klijent.di<<" ";
    if(dajGodine(klijent)<40 && klijent.di==3 && ((klijent.datum[0]>=22 && klijent.datum[1]==12)||(klijent.datum[0]<=12 && klijent.datum[1]==1))){
      cout<<"- Igra lutriju, (di="<<klijent.di<<"), h. znak Jarac, starost "<<dajGodine(klijent);
      cout<<"\nPritisnite bilo sto za nastavak simulacije poslovanja";
      getch();
      cout<<endl;
      }
    cout<<endl;
    Sleep(35);
  }
}

void trudnice(redStrukt *red, short uvjet){
  if(IsEmptyQ(red))
    return;
  kliStrukt pom=FrontQ(red);
  DeQueueQ(red);
  trudnice(red, uvjet);
  if(pom.ci!=3||pom.di!=5)//ako nije trudnica ili ako ne pripada evoTV usluzi - vraćamo je u red
    EnQueueQ(pom, red);
  else
    if(uvjet)
      cout<<"Trudnica "<<pom.ime<<" "<<pom.prezime<<" je cekala za evoTV, sada izlazi iz poslovnice"<<endl;
}

void prepisiRed(redStrukt *izvor, redStrukt *destinacija){
  if(IsEmptyQ(izvor))//rubni
    return;
    
  kliStrukt pom=FrontQ(izvor);
  DeQueueQ(izvor);
  
  prepisiRed(izvor, destinacija);//rek.
  EnQueueQ(pom, izvor);
  EnQueueQ(pom, destinacija);
}

void obrisiRed(redStrukt *red){
  while((!IsEmptyQ(red)))
    DeQueueQ(red);
}

void azurirajGlavni(){
  obrisiRed(glavniRed);
  
  prepisiRed(posiljkaRed, glavniRed);
  prepisiRed(racunRed, glavniRed);
  prepisiRed(lutrijaRed, glavniRed);
  prepisiRed(unionRed, glavniRed);
  prepisiRed(evoTvRed, glavniRed);
  prepisiRed(brziRed, glavniRed);
  
  obrniRed(posiljkaRed);
  obrniRed(racunRed);
  obrniRed(lutrijaRed);
  obrniRed(unionRed);
  obrniRed(evoTvRed);
  obrniRed(brziRed);
}

void dajBrziSalter(redStrukt *red){
  if(IsEmptyQ(red)){//ako smo dosli do kraja reset glob. varijablu na 0
    prvaDva=0;
    return;
  }
  kliStrukt pom=FrontQ(red);
  DeQueueQ(red);
  if(prvaDva<=1){//prva dva stavljamo u brziRed
    EnQueueQ(pom, brziRed); 
    prvaDva++;
  }
  dajBrziSalter(red);
  if(pom.ci==4){//ako se radi o umirovljeniku zapisujemo u brziRed
    EnQueueQ(pom, brziRed); 
  }
  else //Inače vraćamo u red odakle je dosao
    EnQueueQ(pom, red);
}

void brziSalter(){
  brziSim=true;
  dajBrziSalter(evoTvRed);
  dajBrziSalter(unionRed);
  dajBrziSalter(lutrijaRed);
  dajBrziSalter(racunRed);
  dajBrziSalter(posiljkaRed);
  
  obrniRed(posiljkaRed);
  obrniRed(racunRed);
  obrniRed(lutrijaRed);
  obrniRed(unionRed);
  obrniRed(evoTvRed);
  
  sortPrioritet(brziRed,0);
}

bool citajDatoteku(podaci *imeSpol){
  ifstream ulaznaDat("podaciTomislavCivcija.txt", ios::binary);
  if(!ulaznaDat)
    return false;
  char linija [100];
  short poz;
  for(int i=0; i<52; i++){//vadimo podatke
    ulaznaDat.getline(imeSpol[i].ime, 21, '.');
    ulaznaDat.getline(imeSpol[i].prezime, 26, '.');
    ulaznaDat.get(imeSpol[i].spol);
    poz=ulaznaDat.tellg();
    poz+=2;
    ulaznaDat.seekg(poz);
  }
  return true;
}

int main (){
  srand(time(0)%32768);
  podaci* imeSpol=new podaci [52];
  bool procitano=citajDatoteku(imeSpol);
  if(procitano==false){
    cout<<"Datoteka potrebna za rad programa nije na disku! \nProgram ce se sada zavrsiti.";
    Sleep(2000);
    return 0;
  }
  dodajKlijente(imeSpol);
  cout<<"\n/////////////////////////////////\nSimulacija poslovanja je zavrsena\n/////////////////////////////////"<<endl<<endl;
  cout<<"Pritisnite bilo sto za nastavak programa...";
  getch();
  system("cls");
  int izbor;
  
  do {
    cout<<"GLAVNI IZBORNIK: "
      "\n0 Izlaz iz programa"
      "\n1 Izlaz trudnica"
      "\n2 Ispis redova"
      "\n3 Azuriranje glavnog reda"
      "\n4 Simulacija brzog saltera"
      "\n   Izbor: ";
    cin>>izbor;
    switch (izbor){
      case 0: return 0;
      case 1:
        trudnice(glavniRed, 1);
        trudnice(evoTvRed, 0);
        cout<<"Trudnice su izasle zbog predugog cekanja"<<endl;
        break;
      case 2:
        brElemRed=0;
        system("cls");
        cout<<"IZBORNIK ZA ISPIS: "
        "\n   0. Glavni red"
        "\n   1. Posiljka red"
        "\n   2. Racun red"
        "\n   3. Lutrija red"
        "\n   4. West-Union red"
        "\n   5. Evo TV red"
        "\n   6. Brzi red"
        "\n      Izbor: ";
        cin>>izbor;
        cout<<setw(12)<<left<<"Ime"<<setw(13)<<left<<"Prezime"<<setw(6)<<left<<"ai"<<setw(6)<<left<<"bi"<<setw(4)<<left<<"ci"<<setw(4)<<left<<"di"<<endl;
        cout<<setw(12)<<left<<"---"<<setw(13)<<left<<"-------"<<setw(6)<<left<<"--"<<setw(6)<<left<<"--"<<setw(4)<<left<<"--"<<setw(4)<<left<<"--"<<endl;
        switch(izbor) {
          case 0: ispis(glavniRed); obrniRed(glavniRed);break;
          case 1: ispis(posiljkaRed); obrniRed(posiljkaRed); break;
          case 2: ispis(racunRed);obrniRed(racunRed); break;
          case 3: ispis(lutrijaRed);obrniRed(lutrijaRed); break;
          case 4: ispis(unionRed);obrniRed(unionRed); break;
          case 5: ispis(evoTvRed);obrniRed(evoTvRed); break;
          case 6: ispis(brziRed);obrniRed(brziRed); break;     
        }
        cout<<"\n///////KRAJ ISPISA///////"<<endl<<endl;
        cout<<"U redu se nalazi "<<brElemRed<<" osoba"<<endl<<endl;
        break;
      case 3: azurirajGlavni(); break;
      case 4: 
        if(brziSim==true){
          cout<<"Simulacija je vec pokrenuta! Pokrecem ispis reda brzog saltera "<<endl;
          Sleep(2000);
          cout<<setw(12)<<left<<"Ime"<<setw(13)<<left<<"Prezime"<<setw(6)<<left<<"ai"<<setw(6)<<left<<"bi"<<setw(4)<<left<<"ci"<<setw(4)<<left<<"di"<<endl;
          cout<<setw(12)<<left<<"---"<<setw(13)<<left<<"-------"<<setw(6)<<left<<"--"<<setw(6)<<left<<"--"<<setw(4)<<left<<"--"<<setw(4)<<left<<"--"<<endl;
          ispis(brziRed);
          obrniRed(brziRed);
          cout<<"\n///////KRAJ ISPISA///////"<<endl<<endl;
          cout<<"U redu se nalazi "<<brElemRed<<" osoba"<<endl<<endl;
          break;
        }
        brziSalter();
    }
    cout<<"Pritisnite bilo sto za nastavak programa...";
    getch();
    system("cls");
  } while (1);
}
