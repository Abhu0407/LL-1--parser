# LL-1--parser

from tabulate import tabulate

first={}
follow={}
non_terminal=[]
terminal= []
parsing_table={}

def breakrule(rules):
    all=[]
    subrule = ""
    c = 0
    for char in rules:
        if char == ">":
            c = 1
        elif c == 1:
            if char == "|":
                all.append(subrule)
                subrule = ""
            elif char != " ":
                subrule += char
    # print(allrule)
    all.append(subrule)
    return all


def findnonterminal(rulesection):
    for rules in rulesection:
        for c in rules:
            if (c>="A") and (c<="Z"):
                if(non_terminal.count(c)==0):
                    non_terminal.append(c)
            elif (c==" ") or (c=="|") or (c=="-") or (c==">"):
                continue
            else:
                if(terminal.count(c)==0):
                    terminal.append(c)


def removeleftrecursion(rulesection):
    global first,follow,non_terminal,terminal,rules 
    store = []

    for rules in rulesection:
        allrule=[]
        allrule = breakrule(rules)
        

        c = 0
        for r in allrule:
            if r[0] == rules[0]:
                c = 1

        if c == 1:
            pr = rules[0] + "'"
            non_terminal.append(pr)
            alpharule = pr + "-> "
            betarule = rules[0] + "-> "

            for r in allrule:
                if r[0] == rules[0]:
                    alpharule += r[1:] + pr + "|"
                else:
                    betarule += r + pr + "|"

            alpharule += "#"
            if(terminal.count("#")==0):
                terminal.append("#")

            betarule = betarule[:-1]
            store.extend([betarule, alpharule])

        else:
            store.append(rules)

    return store


def removeleftfactorial(rulesection):
    global first,follow,non_terminal,terminal,rules 
    store = []

    for rules in rulesection:
        allrule = []
        allrule = breakrule(rules)
        c = 0
        p = ""

        for i in range(len(allrule)):
            for j in range(i + 1, len(allrule)):
                for k in range(len(allrule[i])):
                    if allrule[i][k] == allrule[j][k]:
                        p += allrule[i][k]
                    else:
                        break
                if p:
                    break
            if p:
                break

        if p:
            x=""
            for c in rules:
                if(c==" ") or (c=="-"):
                    break
                x=x+c
            pr = rules[0] + "'"
            while pr in non_terminal:
                pr += "'"
            non_terminal.append(pr)
            alpharule = x + "-> "
            betarule = pr + "-> "

            n = len(p)
            for r in allrule:
                if r[:n] == p:
                    betarule += r[n:] + "|" if r[n:] else "#|"
                    if(terminal.count("#")==0):
                        terminal.append("#")
                else:
                    alpharule += r + "|"

            alpharule += p + pr
            betarule = betarule[:-1]
            store.extend([alpharule, betarule])
        else:
            store.append(rules)

    return store

def findfirst(rulesection, nt):
    global first, follow, terminal 
    ans = set()  # Initialize ans as a set

    for rules in rulesection:
        if nt == rules[0:len(nt)] and (rules[len(nt)]!="'"):
            #print(rules)
            allrule = []
            allrule = breakrule(rules)
            #print(allrule)
            for r in allrule:
                j = 0
                c = 0
                while c == 0 and j<len(r):
                    if terminal.count(r[j]) > 0:
                        ans.add(r[j])  
                        c = 1
                    else:
                        t = r[j]
                        j = j+1
                        while (j < len(r)) and (r[j] == "'"):
                            t = t + r[j]
                            j = j + 1
                        if len(first.get(t, [])) > 0:
                            check=0
                            for s in first.get(t, []):
                                if s == "#":
                                    check=1
                                else:
                                    ans.add(s)
                            if(check==0):
                                c=1  
                        else:
                            c = 1
                if(j==len(r)) and c==0:
                    ans.add("#")
    return list(ans)  

def findfollow(rulesection, nt):
    global first, follow, non_terminal, terminal, rules
    ans = set()
    if(nt==non_terminal[0]):
        ans.add("$")
    
    for rules in rulesection:
        ft = rules[0]
        i = 1
        while (i < len(rules)) and (rules[i] == "'"):
            ft = ft + rules[i]
            i = i + 1
        allrule = []
        allrule = breakrule(rules)

        for r in allrule:
            i=0
            while i < len(r):
                fnt=r[i]
                i=i+1
                while(i<len(r)) and (r[i]=="'"):
                    fnt=fnt + r[i]
                    i=i+1
                if(fnt==nt):
                    if(i>=len(r)):
                        for element in follow.get(ft, []):
                            ans.add(element)
                        break
                    elif(terminal.count(r[i])>0):
                        ans.add(r[i])
                        break
                    else:
                        c=0
                        while True:
                            if(i>=len(r)):
                                for element in follow.get(ft, []):
                                    ans.add(element)
                                break
                            elif(terminal.count(r[i])>0):
                                ans.add(r[i])
                                break
                            ntr=r[i]
                            i=i+1
                            while(i<len(r)) and (r[i]=="'"):
                                ntr=ntr + r[i]
                                i=i+1
                            check=0
                            for k in first[ntr]:
                                if(k=="#"):
                                    check=1
                                else:
                                    ans.add(k)
                            if(check==0):
                                break

    return list(ans)


def create_parsing_table(rulesection,nt,t):
    global first,follow,non_terminal,terminal,parsing_table
    an=set()
    for rules in rulesection:
        # print(rules)
        if len(nt)<len(rules) and (nt == rules[0:len(nt)] and rules[len(nt)] != "'"):
            allrule = []
            allrule = breakrule(rules)
            for r in allrule:
                i=0
                c=0
                while(i<len(r)) and c==0:
                    if(terminal.count(r[i])>0) and (r[i]==t):
                        an.add(r)
                        break
                    elif(terminal.count(r[i])>0):
                        c=1
                    else:
                        ntr=r[i]
                        i=i+1
                        while(i<len(r)) and r[i]=="'":
                            ntr=ntr+r[i]
                            i=i+1
                        if(first[ntr].count(t)>0):
                            an.add(r)
                            break
                        elif(first[ntr].count("#")>0):
                            c=0
                        else:
                            c=1
            if(first[nt].count("#")>0) and follow[nt].count(t)>0:
                for r in allrule:
                    if(r=="#"):
                        an.add(r)
                        break
                    i=0
                    c=0
                    while(i<len(r)):
                        if(terminal.count(r[i])>0):
                            break
                        else:
                            ntr=r[i]
                            i=i+1
                            while(i<len(r)) and r[i]=="'":
                                ntr=ntr+r[i]
                                i=i+1
                            if(first[ntr].count("#")>0):
                                c=1
                                continue
                            else:
                                c=0
                    if(i==len(r)) and c==1:
                        an.add(r)
    if len(an) == 0:
        an.add("")
    return list(an)

def print_parsing_table(table):
    headers = list(table.keys())
    terminals = set()
    for nt in table.values():
        terminals.update(nt.keys())
    terminals = sorted(list(terminals))
    rows = []
    for nt in headers:
        row = [nt]
        for terminal in terminals:
            productions = table[nt].get(terminal, [])
            row.append(', '.join(productions))
        rows.append(row)

    print(tabulate(rows, headers=[""] + terminals, tablefmt="none"))   



def validateStringUsingStackBuffer(parsing_table, input_string):
    print(f"\nValidate String => {input_string}\n")

    stack = non_terminal[0] +"$" 
    input_string = input_string[::-1]
    buffer = "$"+input_string   # 

    print("{:>20} {:>20} {:>25}".format("Buffer", "Stack", "Action"))
    print("{:>20} {:>20} {:>25}".format(''.join(buffer), ''.join(stack), "-------"))

    while True:
        
        if (stack=="$" and buffer=="$"):
            print("{:>20} {:>20} {:>25}".format('$', '$', "Valid"))
            print("input string are valid")
            return "\nValid String!"
        
        elif(stack=="$" or buffer=="$"):
            print("{:>20} {:>20} {:>25}".format(''.join(buffer), ''.join(stack), "invalid"))
            print("Invalid String!")
            return "\nInvalid String! Unmatched terminal symbols"
        top_stack = stack[0]
        front_buffer = buffer[-1] if buffer else ''

        if top_stack in terminal:  
            if top_stack == front_buffer:  
                print("{:>20} {:>20} {:>25}".format(''.join(buffer), ''.join(stack), f"Matched: {top_stack}"))
                stack=stack[1:]
                buffer=buffer[:-1]
            else:
                return "\nInvalid String! Unmatched terminal symbols"

        else: 
            i=1
            while(i<len(stack) and stack[i]=="'"):
                top_stack=top_stack+stack[i]
                i=i+1
                

            if parsing_table[top_stack][front_buffer]!="": 
                rule = parsing_table[top_stack][front_buffer][0]  
                print("{:>20} {:>20} {:>25}".format(''.join(buffer), ''.join(stack), f" {top_stack} -> {rule}"))
                stack=stack[i:]
                if rule != '#':  
                    stack=rule+stack  
                
            else:
                return f"\nInvalid String! No rule for top of stack: {top_stack}, and input symbol: {front_buffer}."
            




rules=["S -> Ako",
	"A -> Ad | aB | aC",
	"C -> c",
	"B -> bBC | r"]
string="arko"

# rules=["S -> A B  | C",
# 	 "A -> a | b | #",
# 	 "B-> p | #",
# 	 "C -> c"]
# string="acb"

# rules=[
#     "S->ACB|CbB|Ba",
#     "A->da|BC",
#     "B->g|#",
#     "C->h|#"
# ]
# string="abd"

print("Given rules is :- \n")
for s in rules:
    print(s)
print()


findnonterminal(rules)
print("Non terminal :- ",non_terminal)
print("Terminal :- ",terminal)
print()

print("After remove all left recursion in given rules :- \n")
result = removeleftrecursion(rules)
for s in result:
    print(s)  
print()


print("After remove all left factorial in given rules :- \n")
while True:
    a = removeleftfactorial(result)
    if len(a) == len(result):
        break
    result = a

for s in result:
    print(s)
print()


l = 0
while l < 10:
    i = len(non_terminal) - 1
    while i >= 0:
        
        first[non_terminal[i]] = findfirst(result, non_terminal[i])
        
        i = i - 1
    l = l + 1
print("First for all non terminal symbols :- \n")   
for r in non_terminal:
    print(r,":",first[r])
print()


l=0
while(l<10):
    i=len(non_terminal)-1
    while i >= 0:
        follow[non_terminal[i]] = findfollow(result, non_terminal[i])
        i = i - 1
    l=l+1

print("Follow for all non terminal symbols :- \n")
for r in non_terminal:
    print(r,":",follow.get(r, []))
print()

print("Table of First and Follow :- \n ")
table = []
for nt in non_terminal:
    first_set = ', '.join(first.get(nt, []))
    follow_set = ', '.join(follow.get(nt, []))
    table.append([nt, first_set, follow_set])

print(tabulate(table, headers=['Non-terminal', 'First', 'Follow'], tablefmt='none'))

print()
print()

c=0
for nt in non_terminal:
    for t in terminal:
        if(t=="#"):
            t="$"
        if nt not in parsing_table:
            parsing_table[nt] = {}
        if t not in parsing_table[nt]:
            parsing_table[nt][t] = {}

        ans=create_parsing_table(result,nt,t)
        if(len(ans)>1):
            c=1
        parsing_table[nt][t]=ans


print("parsing table :- ")
print_parsing_table(parsing_table)

if(c==1):
    print(" \n Given rules are not in LL(1) parser \n")
else:   
    validateStringUsingStackBuffer(parsing_table, string)
