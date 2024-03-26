# Balckjack writeup :O


## In the following code, we have control of inputing the bet
```c
int betting() //Asks user amount to bet
{
 printf("\n\nEnter Bet: $");
 scanf("%d", &bet);
 
 if (bet > cash) //If player tries to bet more money than player has
 {
        printf("\nYou cannot bet more money than you have.");
        printf("\nEnter Bet: ");
        scanf("%d", &bet);
        return bet;
 }
 else return bet;
} // End Function
```

## and since the 'bet' variable is an int, we could bet negetive amount of money so it will become positive and pass the if conditions.

```c
int betting() //Asks user amount to bet
{
 printf("\n\nEnter Bet: $");
 scanf("%d", &bet);
 
 if (bet > cash) //If player tries to bet more money than player has
 {
        printf("\nYou cannot bet more money than you have.");
        printf("\nEnter Bet: ");
        scanf("%d", &bet);
        return bet;
 }
 else return bet;
} // End Function
```

# here:
```c
    ...
    
    if(dealer_total==21) //Is dealer total is 21, loss
    {
        ...

        cash = cash - bet;

        ...
    } 
    ...
```

# poc:

```bash




              222                111                            
            222 222            11111                              
           222   222          11 111                            
                222              111                               
               222               111                           

CCCCC     SS            DD         HHHHH    C    C                
C    C    SS           D  D       H     H   C   C              
C    C    SS          D    D     H          C  C               
CCCCC     SS          D DD D     H          C C              
C    C    SS         D DDDD D    H          CC C             
C     C   SS         D      D    H          C   C               
C     C   SS        D        D    H     H   C    C             
CCCCCC    SSSSSSS   D        D     HHHHH    C     C            

                        21                                   
     DDDDDDDD      HH         CCCCC    S    S                
        DD        H  H       C     C   S   S              
        DD       H    H     C          S  S               
        DD       H HH H     C          S S              
        DD      H HHHH H    C          SS S             
        DD      H      H    C          S   S               
     D  DD     H        H    C     S   S    C             
      DDD      H        H     CCCCC    S     S            

         222                     111                         
        222                      111                         
       222                       111                         
      222222222222222      111111111111111                       
      2222222222222222    11111111111111111                         


                 Are You Ready?
                ----------------
                      (Y/N)
                        y


Enter 1 to Begin the Greatest Game Ever Played.
Enter 2 to See a Complete Listing of Rules.
Enter 3 to Exit Game. (Not Recommended)
Choice: 1




















Cash: $500
-------
|S    |
|  K  |
|    S|
-------

Your Total is 10

The Dealer Has a Total of 5

Enter Bet: $-1000000


Would You Like to Hit or Stay?
Please Enter H to Hit or S to Stay.
h
-------
|H    |
|  2  |
|    H|
-------

Your Total is 12

The Dealer Has a Total of 15

Would You Like to Hit or Stay?
Please Enter H to Hit or S to Stay.
h
-------
|S    |
|  5  |
|    S|
-------

Your Total is 17

The Dealer Has a Total of 18

Would You Like to Hit or Stay?
Please Enter H to Hit or S to Stay.
s

You Have Chosen to Stay at 17. Wise Decision!

The Dealer Has a Total of 18
Dealer Has the Better Hand. You Lose.

You have 0 Wins and 1 Losses. Awesome!

Would You Like To Play Again?
Please Enter Y for Yes or N for No
y

YaY_I_AM_A_MILLIONARE_LOL


Cash: $1000500
-------
|D    |
|  5  |
|    D|
-------

Your Total is 5

The Dealer Has a Total of 4

Enter Bet: $

```

# the flag: ```YaY_I_AM_A_MILLIONARE_LOL```