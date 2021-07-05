# CitinyCTF
Write-up du Citiny-CTF section cryptanalyse.
## Début
Salut à tous, aujourd'hui je vous présente mon write-up du CitinyCTF présenté dans un prog CLI, c'est un CTF de cryptanalyse et un peu de RE au début.
## Unpacking UPX
La première étape consistait à bypass le ```GetDlgItemText()``` importé par ```user32.dll``` où il nous demande d'insérer un mot de passe pour accéder au CTF. </br><br/>
<img src="https://media.discordapp.net/attachments/736536361258975253/738879854207959133/unknown.png"/><br/><br/>
J'ai donc essayé de voir en quoi le prog était packé, pour ce faire je l'ai d'abord glissé dans ```PEID``` ce qui m'a donné ce résultat.<br/><br/>
<img src="https://media.discordapp.net/attachments/736537536054296636/739248071355007026/unknown.png"/><br/><br/> 
On constate que l'```OEP``` a été modifié par un packer, dorénavant la section de l'```EP``` a été assigné par ```UPX1```. Ce qui veut dire que le programme a été packé en UPX. Désassemblons le programme, pour ma part j'ai utilisé Odbg110, mais rien ne vous empêche d'utiliser un autre désassembleur. Voici ce que nous obtenons.<br/><br/>
<img src="https://media.discordapp.net/attachments/736537536054296636/739254144254214206/unknown.png?width=1786&height=890"/><br/><br/>
Je vais vous expliquer ce que fait ```UPX```, histoire que vous sachiez de quoi on parle. ```UPX``` marche avec un loader qui va chiffrer le code dans l'```EP```. Quand le programme sera exécuté il sera déchiffré. L'```EP``` du loader se trouve dans le ```PUSHAD``` et la routine du déchiffrage se termine au ```POPAD```. J'ai donc désassemblé le programme et je me suis rendu au ```POPAD```. Logiquement, derrière il y aura un ```JMP``` vers l'adresse de l'```EOP```, je mets un breakpoint dans le ```JMP``` vers l'adresse de l'```EOP```, puis je n'avais plus qu'à faire ```F7``` et le tour était joué, je me retrouve dans l'```EOP``` du programme qui là ne sera plus packé. Quelques images pour vous aider à comprendre.<br/><br/>
<img src="https://media.discordapp.net/attachments/736537536054296636/739911499002150912/unknown.png?width=1393&height=343"/><br/><br/>
Dans l'image au dessus, je me suis rendu au ```POPAD``` et j'ai défilé l'instruction ```MOV``` qui a affecté le registre ```ESP``` (registre qui va délimiter la stack).<br/><br/>
<img src="https://media.discordapp.net/attachments/736537536054296636/739914059427610745/unknown.png?width=1225&height=656"/><br/><br/>
Je sélectionne ```ESP```, je fais un clique droit et je clique sur suivre l'interaction du Dump. Ensuite je sélectionne les 4 premiers octets null de ```ESP```, clique droit -> Hardware on access et WORD car ça ne dépasse pas les 4 octets. Ensuite on dump le debugged process<br/><br/>
<img src="https://media.discordapp.net/attachments/740207755410800720/740986429437640804/unknown.png"/><br/><br/>
On positionne l'exécutable et on a plus qu'à la refixer et exécuter le prog.<br/><br/>
<img src="https://media.discordapp.net/attachments/740207755410800720/740986645666463845/unknown.png"/><br/><br/>
Plus qu'à lancer le programme et hop, plus de Popup.<br/><br/>
## Premier Challenge
Voici le premier challenge, lisons et analysons.<br/><br/>
<img src="https://media.discordapp.net/attachments/736537536054296636/739916447031099602/unknown.png?width=751&height=269"/><br/><br/>
La solution se trouvait dans l'indice, si on lit bien on constate qu'il a tenté d'accentuer le mot FairPlay on y ajoutant deux majuscules. 
Mais pourquoi ? Eh bien, j'ai tout simplement inversé les mots Fair et Play ce qui donne "Playfair", ensuite j'ai fait une simple recherche google avec les mots clés "Playfair Cipher" et voici le résultat.<br/><br/>
```Le Chiffre de Playfair ou Carré de Playfair est une méthode manuelle de chiffrement symétrique qui fut la première technique utilisable en pratique de chiffrement par substitution polygrammique. Il fut imaginé en 1854 par Charles Wheatstone, mais porte le nom de Lord Playfair qui popularisa son utilisation.```
<br/></br>
Il s'agit donc du cipher PlayFair, passons au déchiffrage. Pour ce faire, j'ai deux méthodes, soit vous le faites avec un automatisateur de tâche (par ex.: https://www.dcode.fr/chiffre-playfair), soit à la main. Personnellement j'ai opté pour la solution de le faire à la main, c'est bien plus fun. Voici comment j'ai procédé (Lisez biens les règles du Cipher PlayFair).<br/><br/>
<img src="https://media.discordapp.net/attachments/736537536054296636/739922498069594112/unknown.png?width=965&height=242"/><br/><br/>
```YN``` devient ```NO```<br/><br/>
<img src="https://media.discordapp.net/attachments/736537536054296636/739921057284358215/unknown.png?width=960&height=232"/><br/><br/>
```JT``` devient ```PQ```<br/><br/>
<img src="https://media.discordapp.net/attachments/736537536054296636/739921632138625094/unknown.png?width=954&height=231"/><br/><br/>
```AF``` devient ```RS```<br/><br/>
<img src="https://media.discordapp.net/attachments/736537536054296636/739922092195315762/unknown.png?width=961&height=241"/><br/><br/>
```HP``` devient ```TU```<br/><br/>
<img src="https://media.discordapp.net/attachments/736537536054296636/739922734326218762/unknown.png?width=963&height=240"/><br/><br/>
```EK``` devient ```VX```<br/><br/>
<img src="https://media.discordapp.net/attachments/736537536054296636/739923423467274240/unknown.png?width=955&height=238"/><br/><br/>
```DB``` devient ```YZ```<br/><br/>
<img src="https://media.discordapp.net/attachments/736537536054296636/739923787977588918/unknown.png?width=973&height=232"/><br/><br/>
```LY``` devient ```AB```<br/><br/>
<img src="https://media.discordapp.net/attachments/736537536054296636/739924786100174858/unknown.png?width=949&height=246"/><br/><br/>
```IB``` devient ```CD```<br/><br/>
<img src="https://media.discordapp.net/attachments/736537536054296636/739925093471354921/unknown.png?width=951&height=238"/><br/><br/>
```IP``` devient ```EF```<br/><br/>
<img src="https://media.discordapp.net/attachments/736537536054296636/739925534468866179/unknown.png?width=967&height=237"/><br/><br/>
```UG``` devient ```GH```<br/><br/>
<img src="https://media.discordapp.net/attachments/736537536054296636/739925772852133958/unknown.png?width=946&height=230"/><br/><br/>
```CF``` devient ```IJ```<br/><br/>
<img src="https://media.discordapp.net/attachments/736537536054296636/739926056030437466/unknown.png?width=970&height=234"/><br/><br/>
```XA``` devient ```KL```<br/><br/>
<img src="https://media.discordapp.net/attachments/736537536054296636/739926347844943942/unknown.png?width=953&height=240"/><br/><br/>
```QD``` devient ```MB```<br/><br/>
<img src="https://media.discordapp.net/attachments/736537536054296636/739927084884951145/unknown.png?width=399&height=40"/><br/><br/>
Après avoir assemblé tous les résultats, on obtient le résultat final ci-dessus, c'est donc le flag.<br/><br/>
## Deuxième Challenge
Nous voici au second challenge, on a donc bien réussi le premier. Lisons et analysons.<br/><br/>
<img src="https://media.discordapp.net/attachments/740207755410800720/740950935076864051/unknown.png"/><br/><br/>
A priori on voit dans l'indice qu'il s'agit de l'alphabet, si on regarde le Cipher du challenge précédent on constate que c'est l'alphabet mais décalé. Connaissant le chiffrement Caesar je me suis directement mis à compter le nombre de décalage qu'il y avait entre celui ci et le vrai alphabet. La réponse est ```13```, on parle donc du chiffrement ```ROT13```. La méthode de déchiffrage consiste à comparer chaque lettre du message chiffré entre le Cipher et l'alphabet. J'ai donc fait un petit script en python pour automatiser la tâche.<br/><br/>
```py
def citinyctf(n):
    o=""
    key = {
       'a':'n', 'b':'o', 'c':'p', 'd':'q', 'e':'r', 'f':'s', 'g':'t', 'h':'u', 
       'i':'v', 'j':'w', 'k':'x', 'l':'y', 'm':'z', 'n':'a', 'o':'b', 'p':'c', 
       'q':'d', 'r':'e', 's':'f', 't':'g', 'u':'h', 'v':'i', 'w':'j', 'x':'k',
       'y':'l', 'z':'m', 'A':'N', 'B':'O', 'C':'P', 'D':'Q', 'E':'R', 'F':'S', 
       'G':'T', 'H':'U', 'I':'V', 'J':'W', 'K':'X', 'L':'Y', 'M':'Z', 'N':'A', 
       'O':'B', 'P':'C', 'Q':'D', 'R':'E', 'S':'F', 'T':'G', 'U':'H', 'V':'I', 
       'W':'J', 'X':'K', 'Y':'L', 'Z':'M'}
    for x in n:
        v = x in key.keys()
        if v == True:
            o += (key[x])   
        else:
            o += x
    return o

flag = citinyctf("key")
print(flag)
```
On obtient comme résultat "```FTCynitic```", on insère le flag et hop on se retrouve au prochain challenge. Deuxième étape réussie.<br/><br/>
## Troisième Challenge
Nous voisi au troisième challenge, vous connaissez la chanson, lisons et analysons.<br/><br/>
<img src="https://media.discordapp.net/attachments/740207755410800720/740956105244540938/unknown.png"/><br/><br/>
Comme indice on nous dit d'utiliser notre cerveau, je suppose donc qu'on devra trouver le moyen de déchiffrer la key sans sources externes.
Pour résoudre ce challenge j'ai tout simplement comparé chaque lettre par colonnes, la key du challenge précédent était "FTCynitic" et la passphase est "dnourorju".
Je compare donc "```F```" avec "```d```" à travers les colonnes et j'obtiens la lettre "```s```".<br/><br/>
<img src="https://media.discordapp.net/attachments/740207755410800720/740957921025982484/unknown.png"/><br/><br/>
J'ai donc continué sur cette voie.<br/><br/>
<img src="https://media.discordapp.net/attachments/740207755410800720/740958555883962489/unknown.png"/><br/><br/>
<img src="https://media.discordapp.net/attachments/740207755410800720/740959415628333086/unknown.png"/><br/><br/>
J'ai donc fait la même chose pour chaque lettre jusqu'à obtenir le résultat "Seailking". C'est donc le flag final.
Je vous remercie d'avoir lu cette Write-up, il y aura d'autres WriteUPs dans le futur.
