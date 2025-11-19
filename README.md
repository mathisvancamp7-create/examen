import csv

# -------------------------
# NODE KLASSE
# -------------------------
class TaakNode:
    """Representeert één taak in de linked list."""
    def __init__(self, taaknaam, duur, prioriteit):
        self.taaknaam = taaknaam        # Naam van de taak
        self.duur = duur                # Duur in minuten
        self.prioriteit = prioriteit    # Prioriteit (1 = hoogste)
        self.next = None                # Verwijzing naar de volgende node

    def __repr__(self):
        """Mooie representatie van de node voor printen."""
        return f"{self.taaknaam} (duur: {self.duur}, prioriteit: {self.prioriteit})"


# -------------------------
# LINKED LIST KLASSE
# -------------------------
class TaakLinkedList:
    """Representeert de linked list van taken."""
    def __init__(self):
        self.head = None  # Begin van de lijst (head)

    # -------------------------
    # Voeg taak toe achteraan
    # -------------------------
    def add_task(self, taaknaam, duur, prioriteit):
        nieuwe_node = TaakNode(taaknaam, duur, prioriteit)
        # Lijst is leeg, nieuwe node wordt head
        if self.head is None:
            self.head = nieuwe_node
            return
        # Anders loop naar het einde en voeg node toe
        current = self.head
        while current.next:
            current = current.next
        current.next = nieuwe_node

    # -------------------------
    # Verwijder taak op naam
    # -------------------------
    def remove_task(self, taaknaam):
        # Lijst leeg
        if self.head is None:
            return False
        # Verwijder eerste node
        if self.head.taaknaam == taaknaam:
            self.head = self.head.next
            return True
        # Zoek de node om te verwijderen
        prev = self.head
        current = self.head.next
        while current:
            if current.taaknaam == taaknaam:
                prev.next = current.next
                return True
            prev = current
            current = current.next
        return False  # Taak niet gevonden

    # -------------------------
    # Toon alle taken
    # -------------------------
    def display_tasks(self):
        if self.head is None:
            print("Geen taken in de lijst.")
            return
        current = self.head
        while current:
            print(f"{current.taaknaam} — duur: {current.duur}, prioriteit: {current.prioriteit}")
            current = current.next

    # -------------------------
    # Zoek taak op naam
    # -------------------------
    def find_task(self, taaknaam):
        current = self.head
        while current:
            if current.taaknaam == taaknaam:
                return current  # Geeft de node terug
            current = current.next
        return None  # Niet gevonden

    # -------------------------
    # Bereken totale duur van alle taken
    # -------------------------
    def calculate_total_duration(self):
        totaal = 0
        current = self.head
        while current:
            totaal += current.duur
            current = current.next
        return totaal

    # -------------------------
    # Lees taken uit CSV bestand
    # CSV moet kolommen hebben: task_name,duration,priority
    # -------------------------
    def read_tasks_from_csv(self, file_path):
        with open(file_path, newline='') as csvfile:
            reader = csv.DictReader(csvfile)
            for row in reader:
                self.add_task(
                    row["task_name"],
                    int(row["duration"]),
                    int(row["priority"])
                )

    # -------------------------
    # HELPER: insert node in gesorteerde lijst op prioriteit
    # -------------------------
    def sorted_insert_by_priority(self, head, node):
        """
        Voeg node toe in een gesorteerde lijst op prioriteit.
        Lagere prioriteit (1) komt vooraan.
        """
        # Als lijst leeg of node vooraan moet
        if head is None or node.prioriteit < head.prioriteit:
            node.next = head
            return node

        # Zoek juiste plek
        current = head
        while current.next and current.next.prioriteit <= node.prioriteit:
            current = current.next

        # Node invoegen
        node.next = current.next
        current.next = node
        return head

    # -------------------------
    # HELPER: insert node in gesorteerde lijst op prioriteit + duur
    # -------------------------
    def sorted_insert_by_priority_duration(self, head, node):
        """
        Voeg node toe in een gesorteerde lijst:
        eerst op prioriteit, bij gelijke prioriteit op duur.
        """
        # Node moet vooraan
        if (head is None or
            node.prioriteit < head.prioriteit or
            (node.prioriteit == head.prioriteit and node.duur < head.duur)):
            node.next = head
            return node

        # Zoek juiste plek
        current = head
        while current.next and (
            current.next.prioriteit < node.prioriteit or
            (current.next.prioriteit == node.prioriteit and current.next.duur <= node.duur)
        ):
            current = current.next

        # Node invoegen
        node.next = current.next
        current.next = node
        return head

    # -------------------------
    # Sorteer de lijst op prioriteit
    # -------------------------
    def reorder_tasks_by_priority(self):
        """
        Bouw een nieuwe gesorteerde lijst op basis van prioriteit
        en vervang de oude head.
        """
        new_head = None
        current = self.head

        while current:
            next_node = current.next      # Volgende node onthouden
            current.next = None           # Node loskoppelen van oude lijst
            new_head = self.sorted_insert_by_priority(new_head, current)  # Invoegen op juiste plek
            current = next_node           # Ga naar volgende node

        self.head = new_head              # Oude head vervangen door nieuwe gesorteerde lijst

    # -------------------------
    # Sorteer de lijst op prioriteit en daarna duur
    # -------------------------
    def reorder_tasks_by_priority_duration(self):
        """
        Bouw een nieuwe gesorteerde lijst op: eerst prioriteit,
        bij gelijke prioriteit sorteer op duur.
        """
        new_head = None
        current = self.head

        while current:
            next_node = current.next
            current.next = None
            new_head = self.sorted_insert_by_priority_duration(new_head, current)
            current = next_node

        self.head = new_head
