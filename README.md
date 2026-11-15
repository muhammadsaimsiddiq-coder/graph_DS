
# ============================================================
# LIFELINK - BLOOD BANK INVENTORY AND EMERGENCY DONOR MATCHER
# EE-264 Data Structures and Algorithms Course Project
# ============================================================

from datetime import date, timedelta

# ============================================================
# 1. BASIC VARIABLES
# ============================================================

GROUPS = ["O-", "O+", "A-", "A+", "B-", "B+", "AB-", "AB+"]

SHELF_LIFE = 42
ELIGIBILITY_DAYS = 90
LOW_STOCK = 5

URGENCY = {
    1: "CRITICAL",
    2: "URGENT",
    3: "NORMAL"
}


# ============================================================
# 2. HELPER FUNCTIONS
# ============================================================

def check_group(group):
    group = group.strip().upper()

    if group not in GROUPS:
        raise ValueError("Invalid blood group")

    return group


def read_date(value):
    return date.fromisoformat(value)


# ============================================================
# 3. NODE CLASS
# Used by Stack and Queue
# ============================================================

class Node:
    def __init__(self, data):
        self.data = data
        self.next = None


# ============================================================
# 4. STACK
# Used to undo the last operation
# ============================================================

class Stack:
    def __init__(self):
        self.top = None

    def push(self, data):
        new_node = Node(data)
        new_node.next = self.top
        self.top = new_node

    def pop(self):
        if self.top is None:
            return None

        data = self.top.data
        self.top = self.top.next

        return data

    def is_empty(self):
        return self.top is None


# ============================================================
# 5. QUEUE
# Used to manage patient requests
# ============================================================

class Queue:
    def __init__(self):
        self.front = None
        self.rear = None

    def enqueue(self, data):
        new_node = Node(data)

        if self.rear is None:
            self.front = self.rear = new_node
        else:
            self.rear.next = new_node
            self.rear = new_node

    def enqueue_front(self, data):
        new_node = Node(data)
        new_node.next = self.front
        self.front = new_node

        if self.rear is None:
            self.rear = new_node

    def dequeue(self):
        if self.front is None:
            return None

        data = self.front.data
        self.front = self.front.next

        if self.front is None:
            self.rear = None

        return data

    def is_empty(self):
        return self.front is None

    def to_list(self):
        result = []
        current = self.front

        while current:
            result.append(current.data)
            current = current.next

        return result

    def __len__(self):
        count = 0
        current = self.front

        while current:
            count += 1
            current = current.next

        return count


# ============================================================
# 6. AVL TREE NODE
# Stores blood units according to expiry date
# ============================================================
class AVLNode:
    def __init__(self, key, unit):
        self.key = key
        self.unit = unit
        self.left = None
        self.right = None
        self.height = 1



# ============================================================
# 7. AVL TREE
# Insert, Delete and Find Earliest Expiry
# ============================================================

class AVLTree:
    def __init__(self):
        self.root = None
        self.size = 0

    def height(self, node):
        if node is None:
            return 0

        return node.height

    def update_height(self, node):
        node.height = 1 + max(
            self.height(node.left),
            self.height(node.right)
        )

    def balance(self, node):
        return self.height(node.left) - self.height(node.right)

    # Right rotation

    def rotate_right(self, root):
        new_root = root.left
        root.left = new_root.right
        new_root.right = root

        self.update_height(root)
        self.update_height(new_root)

        return new_root

    # Left rotation

    def rotate_left(self, root):
        new_root = root.right
        root.right = new_root.left
        new_root.left = root

        self.update_height(root)
        self.update_height(new_root)

        return new_root

    # Balance AVL Tree

    def rebalance(self, node):
        self.update_height(node)

        balance = self.balance(node)

        if balance > 1:
            if self.balance(node.left) < 0:
                node.left = self.rotate_left(node.left)

            return self.rotate_right(node)

        if balance < -1:
            if self.balance(node.right) > 0:
                node.right = self.rotate_right(node.right)

            return self.rotate_left(node)

        return node

    # Insert a blood unit

    def insert(self, key, unit):
        self.root = self._insert(self.root, key, unit)
        self.size += 1

    def _insert(self, root, key, unit):
        if root is None:
            return AVLNode(key, unit)

        if key < root.key:
            root.left = self._insert(root.left, key, unit)

        elif key > root.key:
            root.right = self._insert(root.right, key, unit)

        else:
            raise ValueError("Duplicate unit")

        return self.rebalance(root)

    # Delete a blood unit

    def delete(self, key):
        self.root, deleted = self._delete(self.root, key)

        if deleted:
            self.size -= 1

        return deleted

    def _delete(self, root, key):
        if root is None:
            return None, False

        if key < root.key:
            root.left, deleted = self._delete(root.left, key)

        elif key > root.key:
            root.right, deleted = self._delete(root.right, key)

        else:
            deleted = True

            if root.left is None:
                return root.right, True

            if root.right is None:
                return root.left, True

            successor = root.right

            while successor.left:
                successor = successor.left

            root.key = successor.key
            root.unit = successor.unit

            root.right, _ = self._delete(
                root.right, successor.key
            )

        if root is None:
            return None, deleted

        return self.rebalance(root), deleted

    # Find the unit with earliest expiry

    def min_unit(self):
        current = self.root

        if current is None:
            return None

        while current.left:
            current = current.left

        return current.unit

    # Remove and return earliest expiry unit

    def pop_min(self):
        unit = self.min_unit()

        if unit is not None:
            self.delete(unit.key())

        return unit

    # Find units expiring within a date range

    def range_query(self, low, high):
        result = []

        def search(node):
            if node is None:
                return

            if low <= node.key[0]:
                search(node.left)

            if low <= node.key[0] <= high:
                result.append(node.unit)

            if node.key[0] <= high:
                search(node.right)

        search(self.root)

        return result

    # Check whether AVL Tree is balanced

    def is_balanced(self):
        def check(node):
            if node is None:
                return True, 0

            left_ok, left_height = check(node.left)
            right_ok, right_height = check(node.right)

            balanced = (
                left_ok
                and right_ok
                and abs(left_height - right_height) <= 1
            )

            height = 1 + max(left_height, right_height)

            return balanced, height

        return check(self.root)[0]


# ============================================================
# 8. BLOOD COMPATIBILITY GRAPH
# Directed graph: Donor -> Recipient
# ============================================================

class BloodGraph:
    def __init__(self):

        self.graph = {
            "O-": ["O-", "O+", "A-", "A+", "B-", "B+", "AB-", "AB+"],
            "O+": ["O+", "A+", "B+", "AB+"],
            "A-": ["A-", "A+", "AB-", "AB+"],
            "A+": ["A+", "AB+"],
            "B-": ["B-", "B+", "AB-", "AB+"],
            "B+": ["B+", "AB+"],
            "AB-": ["AB-", "AB+"],
            "AB+": ["AB+"]
        }

        self.reverse_graph = {g: [] for g in GROUPS}

        for donor in GROUPS:
            for recipient in self.graph[donor]:
                self.reverse_graph[recipient].append(donor)

    # Check blood compatibility

    def can_donate(self, donor, recipient):
        return recipient in self.graph[donor]

    # Find compatible donors

    def donors_for(self, recipient):
        return self.reverse_graph[recipient]

    # Keep O- for last

    def preferred_order(self, recipient):
        donors = self.donors_for(recipient)

        others = [
            donor for donor in donors
            if donor != recipient
        ]

        others.sort(
            key=lambda donor: (
                len(self.graph[donor]),
                donor
            )
        )

        return [recipient] + others


# ============================================================
# 9. AREA GRAPH
# Weighted graph for distance between areas
# ============================================================

class AreaGraph:
    def __init__(self):
        self.graph = {}

    # Add road between two areas

    def add_road(self, area1, area2, distance):
        if distance <= 0:
            raise ValueError("Distance must be positive")

        self.graph.setdefault(area1, {})
        self.graph.setdefault(area2, {})

        self.graph[area1][area2] = distance
        self.graph[area2][area1] = distance

    # Dijkstra shortest path algorithm

    def dijkstra(self, source):
        if source not in self.graph:
            return {}

        distances = {}

        for area in self.graph:
            distances[area] = float("inf")

        distances[source] = 0

        visited = set()

        while len(visited) < len(self.graph):

            current = None
            shortest = float("inf")

            for area in self.graph:
                if area not in visited:
                    if distances[area] < shortest:
                        shortest = distances[area]
                        current = area

            if current is None:
                break

            visited.add(current)

            for neighbour, distance in self.graph[current].items():

                new_distance = distances[current] + distance

                if new_distance < distances[neighbour]:
                    distances[neighbour] = new_distance

        return distances

    def show_areas(self):
        for area in self.graph:
            print(area, ":", self.graph[area])


# ============================================================
# 10. BLOOD UNIT CLASS
# ============================================================

class BloodUnit:
    def __init__(self, unit_id, group, collected, expiry, donor_id=None):
        self.unit_id = unit_id
        self.group = group
        self.collected = collected
        self.expiry = expiry
        self.donor_id = donor_id

    def key(self):
        return (self.expiry.toordinal(), self.unit_id)


# ============================================================
# 11. DONOR CLASS
# ============================================================

class Donor:
    def __init__(self, donor_id, name, group, area, last_donation=None):
        self.donor_id = donor_id
        self.name = name
        self.group = group
        self.area = area
        self.last_donation = last_donation


# ============================================================
# 12. REQUEST CLASS
# ============================================================

class Request:
    def __init__(self, req_id, patient, group, units, urgency, area):
        self.req_id = req_id
        self.patient = patient
        self.group = group
        self.units = units
        self.urgency = urgency
        self.area = area


# ============================================================
# 13. RESULT CLASS
# ============================================================

class Result:
    def __init__(self, request):
        self.request = request
        self.issued = []
        self.shortfall = 0
        self.donors = []


# ============================================================
# 14. MAIN BLOOD BANK CLASS
# Connects all data structures
# ============================================================

class BloodBank:

    def __init__(self):

        self.today = date.today()

        self.blood_graph = BloodGraph()
        self.area_graph = AreaGraph()

        # Separate AVL Tree for every blood group

        self.trees = {
            group: AVLTree()
            for group in GROUPS
        }

        self.units = {}
        self.donors = {}

        # Donor IDs stored according to blood group

        self.donors_by_group = {
            group: set()
            for group in GROUPS
        }

        # Three queues according to urgency

        self.queues = {
            1: Queue(),
            2: Queue(),
            3: Queue()
        }

        self.undo_stack = Stack()

        self.log = []

        self.discarded = []

        self.thresholds = {
            group: LOW_STOCK
            for group in GROUPS
        }

        self.unit_count = 0
        self.donor_count = 0
        self.request_count = 0

    # --------------------------------------------------------
    # Activity Log
    # --------------------------------------------------------

    def add_log(self, message):
        self.log.append(str(self.today) + " : " + message)

        if len(self.log) > 100:
            self.log.pop(0)

    # --------------------------------------------------------
    # Set Today's Date
    # --------------------------------------------------------

    def set_today(self, new_date):
        self.today = read_date(new_date)
        self.remove_expired()
        self.add_log("Date updated")

    # --------------------------------------------------------
    # Total Blood Units
    # --------------------------------------------------------

    def total_units(self):
        return len(self.units)

    # --------------------------------------------------------
    # Count Blood Group Units
    # --------------------------------------------------------

    def count(self, group):
        return self.trees[group].size

    # --------------------------------------------------------
    # ADD BLOOD UNIT
    # --------------------------------------------------------

    def add_unit(self, group, collected=None, donor_id=None):

        group = check_group(group)

        if collected is None:
            collected = self.today

        if isinstance(collected, str):
            collected = read_date(collected)

        if collected > self.today:
            raise ValueError("Collection date cannot be in future")

        expiry = collected + timedelta(days=SHELF_LIFE)

        if expiry < self.today:
            raise ValueError("Blood unit is already expired")

        if donor_id is not None and donor_id not in self.donors:
            raise ValueError("Donor not found")

        self.unit_count += 1

        unit_id = "U" + str(self.unit_count).zfill(4)

        unit = BloodUnit(
            unit_id, group, collected, expiry, donor_id
        )

        self.trees[group].insert(unit.key(), unit)

        self.units[unit_id] = unit

        previous_date = None

        if donor_id is not None:
            donor = self.donors[donor_id]
            previous_date = donor.last_donation
            donor.last_donation = collected

        self.undo_stack.push(
            ("ADD", unit, previous_date)
        )

        self.add_log("Added " + unit_id + " (" + group + ")")

        print("Blood unit added:", unit_id)
        print("Expiry date:", expiry)

        return unit

    # --------------------------------------------------------
    # REMOVE EXPIRED BLOOD UNITS
    # --------------------------------------------------------

    def remove_expired(self):

        expired_units = []

        for group in GROUPS:

            tree = self.trees[group]

            while True:

                unit = tree.min_unit()

                if unit is None:
                    break

                if unit.expiry >= self.today:
                    break

                tree.pop_min()

                del self.units[unit.unit_id]

                expired_units.append(unit)

        if expired_units:
            self.discarded.extend(expired_units)

            self.add_log(
                "Discarded " + str(len(expired_units)) + " expired units"
            )

        return expired_units

    # --------------------------------------------------------
    # EXPIRING SOON REPORT
    # --------------------------------------------------------

    def expiring_soon(self, days=7):

        self.remove_expired()

        low = self.today.toordinal()
        high = (self.today + timedelta(days=days)).toordinal()

        result = []

        for group in GROUPS:
            result.extend(
                self.trees[group].range_query(low, high)
            )

        result.sort(
            key=lambda unit: (unit.expiry, unit.unit_id)
        )

        return result

    # --------------------------------------------------------
    # REGISTER DONOR
    # --------------------------------------------------------

    def register_donor(
        self, name, group, area, last_donation=None
    ):

        group = check_group(group)

        if not name.strip():
            raise ValueError("Name cannot be empty")

        if last_donation is not None:
            if isinstance(last_donation, str):
                last_donation = read_date(last_donation)

            if last_donation > self.today:
                raise ValueError("Donation date cannot be in future")

        self.donor_count += 1

        donor_id = "D" + str(self.donor_count).zfill(3)

        donor = Donor(
            donor_id, name, group, area, last_donation
        )

        self.donors[donor_id] = donor

        self.donors_by_group[group].add(donor_id)

        self.add_log("Registered donor " + donor_id)

        print("Donor registered:", donor_id)

        return donor_id

    # --------------------------------------------------------
    # CHECK DONOR ELIGIBILITY
    # --------------------------------------------------------

    def is_eligible(self, donor):

        if donor.last_donation is None:
            return True

        days = (self.today - donor.last_donation).days

        return days >= ELIGIBILITY_DAYS

    # --------------------------------------------------------
    # FIND COMPATIBLE DONORS
    # Nearest donors are shown first
    # --------------------------------------------------------

    def find_donors(self, group, area=None, limit=10):

        group = check_group(group)

        distances = self.area_graph.dijkstra(area) if area else {}

        donor_groups = self.blood_graph.preferred_order(group)

        candidates = []

        for group_rank, donor_group in enumerate(donor_groups):

            for donor_id in self.donors_by_group[donor_group]:

                donor = self.donors[donor_id]

                if self.is_eligible(donor):

                    distance = distances.get(
                        donor.area, float("inf")
                    )

                    candidates.append(
                        (donor, distance, group_rank)
                    )

        candidates.sort(
            key=lambda item: (
                item[2],
                item[1],
                item[0].donor_id
            )
        )

        return [
            (donor, distance)
            for donor, distance, rank in candidates[:limit]
        ]

    # --------------------------------------------------------
    # SUBMIT PATIENT REQUEST
    # --------------------------------------------------------

    def submit_request(
        self, patient, group, units, urgency, area=""
    ):

        group = check_group(group)

        if not patient.strip():
            raise ValueError("Patient name cannot be empty")

        if not isinstance(units, int) or units < 1:
            raise ValueError("Enter valid number of units")

        if urgency not in URGENCY:
            raise ValueError("Invalid urgency")

        self.request_count += 1

        request = Request(
            "R" + str(self.request_count).zfill(3),
            patient,
            group,
            units,
            urgency,
            area
        )

        self.queues[urgency].enqueue(request)

        self.add_log("Submitted request " + request.req_id)

        print("Request submitted:", request.req_id)

        return request

    # --------------------------------------------------------
    # SHOW PENDING REQUESTS
    # --------------------------------------------------------

    def pending(self):

        result = []

        for urgency in [1, 2, 3]:
            result.extend(self.queues[urgency].to_list())

        return result

    # --------------------------------------------------------
    # PROCESS NEXT REQUEST
    # Critical -> Urgent -> Normal
    # FIFO inside each queue
    # --------------------------------------------------------

    def process_next(self):

        request = None

        for urgency in [1, 2, 3]:

            if not self.queues[urgency].is_empty():

                request = self.queues[urgency].dequeue()
                break

        if request is None:
            print("No pending requests")
            return None

        self.remove_expired()

        result = Result(request)

        needed = request.units

        donor_order = self.blood_graph.preferred_order(
            request.group
        )

        # Exact match first, then compatible groups

        for group in donor_order:

            tree = self.trees[group]

            while needed > 0 and tree.size > 0:

                unit = tree.pop_min()

                del self.units[unit.unit_id]

                result.issued.append(unit)

                needed -= 1

            if needed == 0:
                break

        result.shortfall = needed

        # Find eligible donors if stock is insufficient

        if needed > 0:

            result.donors = self.find_donors(
                request.group,
                request.area
            )

        # Save action for Undo

        self.undo_stack.push(
            ("PROCESS", request, result.issued)
        )

        self.add_log(
            "Processed request " + request.req_id
        )

        # Display result

        print("\nREQUEST RESULT")
        print("Patient:", request.patient)
        print("Required:", request.units)
        print("Issued:", len(result.issued))

        for unit in result.issued:

            print(
                unit.unit_id,
                unit.group,
                "Expiry:",
                unit.expiry
            )

        if result.shortfall > 0:

            print("Shortfall:", result.shortfall)

            print("\nEligible donor suggestions:")

            for donor, distance in result.donors:

                if distance == float("inf"):
                    distance_text = "Unknown"
                else:
                    distance_text = str(distance) + " km"

                print(
                    donor.donor_id,
                    donor.name,
                    donor.group,
                    donor.area,
                    distance_text
                )

        return result

    # --------------------------------------------------------
    # PROCESS ALL REQUESTS
    # --------------------------------------------------------

    def process_all(self):

        results = []

        while True:

            result = self.process_next()

            if result is None:
                break

            results.append(result)

        return results

    # --------------------------------------------------------
    # UNDO LAST ACTION
    # --------------------------------------------------------

    def undo(self):

        action = self.undo_stack.pop()

        if action is None:
            print("Nothing to undo")
            return

        if action[0] == "ADD":

            unit = action[1]
            previous_date = action[2]

            self.trees[unit.group].delete(unit.key())

            del self.units[unit.unit_id]

            if unit.donor_id is not None:
                self.donors[
                    unit.donor_id
                ].last_donation = previous_date

            message = "Blood addition undone"

        elif action[0] == "PROCESS":

            request = action[1]
            issued_units = action[2]

            # Return issued units to inventory

            for unit in issued_units:

                self.trees[unit.group].insert(
                    unit.key(), unit
                )

                self.units[unit.unit_id] = unit

            # Put request back at the front of its queue

            self.queues[
                request.urgency
            ].enqueue_front(request)

            message = "Request processing undone"

        self.add_log(message)

        print(message)

    # --------------------------------------------------------
    # STOCK REPORT
    # --------------------------------------------------------

    def stock_report(self):

        print("\n========== STOCK REPORT ==========")

        for group in GROUPS:

            count = self.count(group)

            earliest = self.trees[group].min_unit()

            if earliest:
                expiry = earliest.expiry
            else:
                expiry = "No stock"

            status = ""

            if count < self.thresholds[group]:
                status = " LOW STOCK"

            print(
                group,
                "| Units:", count,
                "| Earliest expiry:", expiry,
                status
            )

        print("Total units:", self.total_units())

    # --------------------------------------------------------
    # COMPATIBLE STOCK
    # --------------------------------------------------------

    def compatible_stock(self, recipient):

        recipient = check_group(recipient)

        result = {}

        for group in self.blood_graph.donors_for(recipient):

            result[group] = self.count(group)

        return result

    # --------------------------------------------------------
    # LOW STOCK REPORT
    # --------------------------------------------------------

    def low_stock(self):

        result = []

        for group in GROUPS:

            if self.count(group) < self.thresholds[group]:
                result.append(group)

        return result

    # --------------------------------------------------------
    # RECENT ACTIVITY
    # --------------------------------------------------------

    def show_log(self):

        print("\n========== RECENT ACTIVITY ==========")

        if not self.log:
            print("No activity yet")
            return

        for entry in self.log:
            print(entry)

    # --------------------------------------------------------
    # AVL TREE STATUS
    # --------------------------------------------------------

    def check_trees(self):

        for group in GROUPS:

            balanced = self.trees[group].is_balanced()

            print(group, ":", "Balanced" if balanced else "Not Balanced")


# ============================================================
# 15. SAMPLE AREA DATA
# Karachi areas and sample distances
# ============================================================

def load_roads(bank):

    roads = [
        ("Gulshan", "North Nazimabad", 7),
        ("Gulshan", "Saddar", 11),
        ("Gulshan", "Malir", 12),
        ("Gulshan", "Korangi", 14),
        ("Saddar", "Clifton", 6),
        ("Saddar", "Lyari", 5),
        ("Clifton", "DHA", 4),
        ("DHA", "Korangi", 9),
        ("North Nazimabad", "Orangi", 8),
        ("North Nazimabad", "Lyari", 10),
        ("Malir", "Landhi", 7),
        ("Landhi", "Korangi", 6)
    ]

    for area1, area2, distance in roads:

        bank.area_graph.add_road(
            area1, area2, distance
        )


# ============================================================
# 16. DEMO DATA
# ============================================================

def load_demo(bank):

    load_roads(bank)

    counts = {
        "O-": 3,
        "O+": 8,
        "A-": 2,
        "A+": 7,
        "B-": 2,
        "B+": 6,
        "AB-": 1,
        "AB+": 2
    }

    # Add sample blood units

    for group in GROUPS:

        for i in range(counts[group]):

            collected = bank.today - timedelta(
                days=(7 * i) % 38
            )

            bank.add_unit(group, collected)

    # Register sample donors

    people = [
        ("Ali Raza", "O-", "Gulshan", 120),
        ("Sara Khan", "A+", "Clifton", 200),
        ("Bilal Ahmed", "B+", "Saddar", 30),
        ("Hina Malik", "AB-", "Malir", None),
        ("Usman Ali", "O+", "DHA", 95),
        ("Ayesha Noor", "A-", "Korangi", 150),
        ("Zain Abbas", "B-", "Orangi", 400),
        ("Fatima Sheikh", "O+", "Landhi", 60),
        ("Hamza Iqbal", "A+", "Lyari", None),
        ("Maryam Aslam", "AB+", "North Nazimabad", 180)
    ]

    for name, group, area, days_ago in people:

        if days_ago is None:
            last_date = None
        else:
            last_date = bank.today - timedelta(
                days=days_ago
            )

        bank.register_donor(
            name, group, area, last_date
        )

    # Demo loading should not be undone

    bank.undo_stack = Stack()

    print("Demo data loaded successfully")


# ============================================================
# 17. DISPLAY MENU
# ============================================================

MENU = """
============== LIFELINK BLOOD BANK ==============

1. Add Blood Unit
2. Register Donor
3. Submit Patient Request
4. Process Next Request
5. Process All Requests
6. Stock Report
7. Expiring Soon Report
8. Find Eligible Donors
9. Blood Compatibility Lookup
10. Undo Last Action
11. Set Today's Date
12. Recent Activity Log
13. Load Demo Data
14. Add Road Between Areas
15. Show Areas
16. Check AVL Trees
0. Exit

=================================================
"""


# ============================================================
# 18. MAIN PROGRAM
# ============================================================

def main():

    bank = BloodBank()

    print("Welcome to LifeLink Blood Bank")
    print("Educational Prototype - Not for Clinical Use")

    while True:

        print(MENU)

        print("Today:", bank.today)
        print("Total Blood Units:", bank.total_units())
        print("Pending Requests:", len(bank.pending()))

        try:

            choice = input("\nEnter your choice: ").strip()

            # Add blood unit

            if choice == "1":

                group = input("Enter blood group: ")

                donor_id = input(
                    "Enter donor ID (press Enter to skip): "
                ).strip()

                if donor_id == "":
                    donor_id = None

                collected = input(
                    "Collection date (YYYY-MM-DD, Enter for today): "
                ).strip()

                if collected == "":
                    collected = bank.today

                bank.add_unit(
                    group, collected, donor_id
                )

            # Register donor

            elif choice == "2":

                name = input("Enter donor name: ")
                group = input("Enter blood group: ")
                area = input("Enter area: ")

                last = input(
                    "Last donation date (YYYY-MM-DD, Enter if none): "
                ).strip()

                if last == "":
                    last = None

                donor_id = bank.register_donor(
                    name, group, area, last
                )

                print("Donor ID:", donor_id)

            # Submit patient request

            elif choice == "3":

                patient = input("Enter patient name: ")
                group = input("Required blood group: ")

                units = int(
                    input("Number of units: ")
                )

                print("1. Critical")
                print("2. Urgent")
                print("3. Normal")

                urgency = int(
                    input("Select urgency: ")
                )

                area = input("Patient area: ")

                bank.submit_request(
                    patient,
                    group,
                    units,
                    urgency,
                    area
                )

            # Process next request

            elif choice == "4":

                bank.process_next()

            # Process all requests

            elif choice == "5":

                bank.process_all()

            # Stock report

            elif choice == "6":

                bank.remove_expired()
                bank.stock_report()

                print("Low stock groups:", bank.low_stock())

            # Expiring soon

            elif choice == "7":

                days = int(
                    input("Show units expiring within how many days? ")
                )

                result = bank.expiring_soon(days)

                if not result:
                    print("No units expiring in this period")

                for unit in result:

                    print(
                        unit.unit_id,
                        unit.group,
                        "Expiry:",
                        unit.expiry
                    )

            # Find eligible donors

            elif choice == "8":

                group = input("Required blood group: ")
                area = input("Patient area: ")

                donors = bank.find_donors(group, area)

                if not donors:
                    print("No eligible donors found")

                for donor, distance in donors:

                    if distance == float("inf"):
                        distance = "Unknown"
                    else:
                        distance = str(distance) + " km"

                    print(
                        donor.donor_id,
                        donor.name,
                        donor.group,
                        donor.area,
                        distance
                    )

            # Compatibility lookup

            elif choice == "9":

                group = input("Enter patient blood group: ")

                compatible = bank.compatible_stock(group)

                print("\nCompatible blood stock:")

                for donor_group, count in compatible.items():

                    print(donor_group, ":", count)

            # Undo

            elif choice == "10":

                bank.undo()

            # Set date

            elif choice == "11":

                new_date = input(
                    "Enter date (YYYY-MM-DD): "
                )

                bank.set_today(new_date)

                print("Date updated:", bank.today)

            # Activity log

            elif choice == "12":

                bank.show_log()

            # Load demo data

            elif choice == "13":

                load_demo(bank)

            # Add road

            elif choice == "14":

                area1 = input("First area: ")
                area2 = input("Second area: ")

                distance = float(
                    input("Distance in km: ")
                )

                bank.area_graph.add_road(
                    area1, area2, distance
                )

                print("Road added successfully")

            # Show areas

            elif choice == "15":

                bank.area_graph.show_areas()

            # Check AVL balance

            elif choice == "16":

                bank.check_trees()

            # Exit

            elif choice == "0":

                print("Thank you for using LifeLink")
                break

            else:

                print("Invalid choice")

        except (ValueError, KeyError) as error:

            print("Error:", error)

        except Exception as error:

            print("Unexpected error:", error)


# ============================================================
# 19. START PROGRAM
# ============================================================

if __name__ == "__main__":
    main()

