# android2
# Analiza cyklu życia aplikacji TodoApp

## Przegląd architektury aplikacji

Aplikacja TodoApp składa się z:
- **TaskListActivity** - główna aktywność z listą zadań
- **MainActivity** - aktywność szczegółów zadania  
- **SingleFragmentActivity** - abstrakcyjna klasa bazowa
- **TaskListFragment** - fragment wyświetlający listę zadań
- **TaskFragment** - fragment edycji pojedynczego zadania
- **TaskStorage** - Singleton zarządzający danymi

## Cykl życia aktywności

### 1. TaskListActivity (aktywność główna)

#### Rozpoczęcie aplikacji:
```
onCreate() → onStart() → onResume() → [AKTYWNA]
```

**onCreate():**
- Dziedziczenie po `SingleFragmentActivity`
- Wywołanie `createFragment()` zwracające `TaskListFragment`
- Inicjalizacja układu `activity_single_fragment.xml`
- Dodanie fragmentu do kontenera za pomocą `FragmentManager`

**onStart():**
- Aktywność staje się widoczna dla użytkownika
- Fragment `TaskListFragment` również przechodzi przez `onStart()`

**onResume():**
- Aktywność jest w pełni interaktywna
- Fragment wywołuje `onResume()` → `updateView()`
- **Kluczowe:** Lista zadań jest odświeżana z `TaskStorage`

### 2. MainActivity (aktywność szczegółów zadania)

#### Przejście z listy do szczegółów:
```
[Kliknięcie zadania na liście]
TaskListActivity: onPause() → onStop()
MainActivity: onCreate() → onStart() → onResume() → [AKTYWNA]
```

**onCreate():**
- Pobieranie `taskId` z `Intent.getExtras()`
- Wywołanie `createFragment()` z parametrem `taskId`
- Tworzenie `TaskFragment` dla konkretnego zadania
- Ładowanie danych zadania z `TaskStorage`

**onCreateView() w TaskFragment:**
- Inicjalizacja pól formularza
- Ustawienie wartości z obiektu `Task`
- Konfiguracja listenerów dla przycisków i pól tekstowych

### 3. Powrót do listy zadań

#### Nawigacja wstecz:
```
[Naciśnięcie przycisku Back lub Up]
MainActivity: onPause() → onStop() → onDestroy()
TaskListActivity: onRestart() → onStart() → onResume()
TaskListFragment: onResume() → updateView()
```

## Cykl życia fragmentów

### TaskListFragment

**onCreate():**
- Inicjalizacja bez tworzenia obiektu `Task`
- Konfiguracja adaptera `TaskAdapter`

**onCreateView():**
- Powiązanie z layoutem `fragment_task_list.xml`
- Inicjalizacja `RecyclerView`
- Ustawienie `LinearLayoutManager`
- Wywołanie `updateView()` na końcu

**onResume():**
- **Kluczowy moment:** Ponowne wywołanie `updateView()`
- Zapewnia synchronizację danych po powrocie z edycji

**updateView():**
- Pobieranie aktualnej listy z `TaskStorage.getInstance().getTasks()`
- Aktualizacja adaptera `notifyDataSetChanged()`

### TaskFragment

**onCreate():**
- Pobieranie `taskId` z argumentów fragmentu
- Ładowanie konkretnego zadania z `TaskStorage.getInstance().getTask(taskId)`

**onCreateView():**
- Inicjalizacja kontrolek (TextView, EditText, Button, CheckBox)
- Ustawienie wartości pól na podstawie danych zadania:
  - `taskNameEditText.setText(task.getName())`
  - `taskDetailsEditText.setText(task.getDetails())`
  - `taskDoneCheckBox.setChecked(task.isDone())`
- Konfiguracja listenerów dla edycji

## Mechanizmy zarządzania stanem

### TaskStorage (Singleton)

**Zalety:**
- **Centralne zarządzanie danymi** - jeden punkt dostępu
- **Przetrwanie między aktywnościami** - dane nie giną przy przejściach
- **Synchronizacja automatyczna** - zmiany widoczne natychmiast

**Metody kluczowe:**
- `getTasks()` - zwraca całą listę zadań
- `getTask(UUID id)` - zwraca konkretne zadanie
- Automatyczne generowanie 100+ przykładowych zadań w konstruktorze

### Przepływ danych

```
1. TaskListActivity → wyświetla listę z TaskStorage
2. Kliknięcie zadania → Intent z taskId → MainActivity  
3. TaskFragment → edytuje zadanie w TaskStorage
4. Powrót → TaskListFragment.onResume() → updateView() → odświeżona lista
```

## Kluczowe wzorce projektowe

### 1. Single Activity Pattern (częściowo)
- Wykorzystanie fragmentów zamiast wielu aktywności
- `SingleFragmentActivity` jako abstrakcyjna klasa bazowa

### 2. Adapter Pattern
- `TaskAdapter` implementuje `RecyclerView.Adapter`
- `TaskHolder` implementuje `ViewHolder` pattern

### 3. Singleton Pattern
- `TaskStorage` zapewnia globalny dostęp do danych
- Lazy initialization w `getInstance()`

### 4. Template Method Pattern
- `SingleFragmentActivity` definiuje szkielet `onCreate()`
- Konkretne aktywności implementują `createFragment()`

## Obsługa zdarzeń lifecycle

### Kluczowe implementacje:

**onResume() w TaskListFragment:**
```java
@Override
public void onResume() {
    super.onResume();
    updateView(); // Odświeżenie listy po powrocie
}
```

**onClick() w TaskHolder:**
```java
@Override
public void onClick(View view) {
    Intent intent = new Intent(getActivity(), MainActivity.class);
    intent.putExtra("taskId", task.getId().toString());
    startActivity(intent);
}
```

## Zalety tej implementacji

### 1. Automatyczna synchronizacja danych
- `onResume()` zapewnia aktualizację po każdym powrocie
- Brak potrzeby ręcznego odświeżania

### 2. Separacja odpowiedzialności
- Model (Task) oddzielony od widoku
- Storage zarządza tylko danymi
- Fragmenty odpowiadają tylko za UI

### 3. Elastyczność
- Łatwe dodawanie nowych fragmentów
- Uniwersalny kontener w `activity_single_fragment.xml`
- Możliwość rozbudowy bez zmian w klasach bazowych

### 4. Wydajność
- RecyclerView z ViewHolder pattern
- Ponowne użycie widoków elementów listy

## Potencjalne usprawnienia

### 1. Obsługa orientacji ekranu
- Zapisywanie stanu fragmentów w `onSaveInstanceState()`
- Przywracanie stanu w `onCreate()`

### 2. Persistence danych
- Zamiana Singleton na bazę danych (Room)
- Zapisywanie zmian na dysku

### 3. Navigation Component
- Zamiana Intent na Navigation Graph
- Lepsze zarządzanie back stack

### 4. ViewModel + LiveData
- Separation of concerns zgodnie z MVVM
- Automatyczne reagowanie na zmiany danych

## Podsumowanie

Aplikacja TodoApp prawidłowo wykorzystuje cykl życia Android do:
- **Zarządzania stanem** między przejściami
- **Synchronizacji danych** między ekranami  
- **Optymalizacji wydajności** poprzez właściwe lifecycle callbacks

Kluczowym elementem jest wykorzystanie `onResume()` w `TaskListFragment` do automatycznego odświeżania listy, co zapewnia spójność danych bez dodatkowego kodu.
