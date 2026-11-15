# graph_DS
class Friendship:
    def __init__(self, target, weight=1):
        self.target=target
        self.weight=weight
class User:
    def __init__(self,name):
        self.name = name
        self.neighbors = []
    def add_neighbor(self, node, weight=1):
        friendship = Friendship(node, weight)
        self.neighbors.append(friendship)
    def show_friends(self):
        x = len(self.neighbors)
        print("Friends of",self.name,"-->",end=" ")
        for i in range(x):
            print(self.neighbors[i].target.name,end="")
            if i != x-1:
                print(",",end="")
            if i == x-1:
                print()

class Social_Network:
    def __init__(self):
         self.nodes=[]
    def add_user(self, name):
        new_node=User(name)
        self.nodes.append(new_node)
        print("User",name,"Added.")
        return new_node
    def add_edge(self,u,v,directed,weight=1): #sourcenode(u), targetnode(v)
        u.add_neighbor(v,weight)
        if not directed:
            v.add_neighbor(u,weight)
        print("Friendship created:",u.name,"<-->",v.name)
    def display(self):
        print("Social Network:")
        n=len(self.nodes)
        for node in self.nodes:
            node.show_friends()
        print()

G = Social_Network()
G.add_user('Amir')
G.add_user('Bilal')
G.add_user('Zohaib')
G.add_user('Ahmad')
G.add_edge(G.nodes[0],G.nodes[1],"")
G.add_edge(G.nodes[1],G.nodes[2],"")
G.add_edge(G.nodes[3],G.nodes[2],"")
G.nodes[1].show_friends()
G.display()


        
