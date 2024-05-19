# Tic_Tac_Toe_-P-_5.0-
#include <stdio.h>  
#include <conio.h>
#include <stdbool.h> 
#include<stdlib.h>
#include<time.h>

#define win 100
#define threat 101
#define O 111
#define X 120
#define nothing 102
#define loose 103
#define draw 104
#define number 100000000

//We set up a structure to make all the formulas into the computer player and the overall situation clearer

struct place {
    int all;
    int person;
    int computer;
};
//Zero the function that records the score
void reset (int *mark){
    for (int i=0; i<8; i++){
        mark[i]=0;
    }
}
//Determine if the player wins
void check_win_person (const struct place (*P)[3], int *mark){
    int x=0 , y=0 , sum=0 , i=0;
    reset(mark);

    for (x=0;x<3;x++){
        for (y=0;y<3;y++){
            sum += P[y][x].person;
        }
        mark[i]=sum;
        i++;sum=0;
    }

    for (y=0;y<3;y++){
        for (x=0;x<3;x++){
            sum += P[y][x].person;
        }
        mark[i]=sum;
        i++;sum=0;
    }


    for (int n=0 ; n<3 ; n++){
        sum +=P[n][n].person;    
    }
    mark[i]=sum;
    i++;sum=0;

    for (int n=0 ; n<3 ; n++){
        sum +=P[n][2-n].person;    
    }
    mark[i]=sum;
    i++;sum=0;
}
//Determine whether the computer wins
void check_win_computer (const struct place (*P)[3], int *mark){
    int x=0 , y=0 , sum=0 , i=0;
    reset(mark);

    for (x=0;x<3;x++){
        for (y=0;y<3;y++){
            sum += P[y][x].computer;
        }
        mark[i]=sum;
        i++;sum=0;
    }

    for (y=0;y<3;y++){
        for (x=0;x<3;x++){
            sum += P[y][x].computer;
        }
        mark[i]=sum;
        i++;sum=0;
    }


    for (int n=0 ; n<3 ; n++){
        sum +=P[n][n].computer;    
    }
    mark[i]=sum;
    i++;sum=0;

    for (int n=0 ; n<3 ; n++){
        sum +=P[n][2-n].computer;    
    }
    mark[i]=sum;
    i++;sum=0;
}
//Restore all data in the structure to its original state
void fth_data(struct place (*P)[3]){
    for(int x = 0; x < 3; x++) { 
        for(int y = 0; y < 3; y++) {  
            P[y][x].all = 46;  
            P[y][x].computer = 0;   
            P[y][x].person = 0; 
        }  
    }     
}
//redraw
void re_print(const struct place (*P)[3]){
    system("cls");
    for(int y = 0; y < 3; y++) {   
        for(int x = 0; x < 3; x++) {   
            printf("%c\t", P[y][x].all);  
        }  
        printf("\n");   
    }    
}
//Converts a single number into two coordinates
void tranxy (int *nx, int *ny, int n){
    *ny = 3 - ((n + 2) / 3);
    *nx = (n + 2) % 3;
}
//Determine if this point has already been hit by the player or the computer
bool check_exist(const struct place (*P)[3], int n){
    int nx, ny;
    tranxy(&nx,&ny,n);

    if (P[ny][nx].all==46){
        return true;//can be put
    }else{
        return false;
    }
}
//All the points are counted to determine whether they are a victory or a threat
int score (const int *mark){
    int res;
    int L=8, num=0;

    for (int i=0 ; i<L ; i++){
        if (mark[i]==3){
            res=win;
            break;
        }else if (mark[i]==2){
            num++;
        }
    }
    
    if (num>=2 && res!=win){
        res=threat;
    }

    return res ;
}
//ai brain for avoiding loosing
int ai_pre_step_one(const struct place P[3][3], int *mark){
    struct place P_f[3][3];
    int nx, ny, res=nothing;

    for(int n=1 ; n<=9 ; n++ ){

        if(check_exist(P,n)){
            for(int x = 0; x < 3; x++) { 
                for(int y = 0; y < 3; y++) {  
                    P_f[y][x].computer = P[y][x].computer;
                    P_f[y][x].person = P[y][x].person;   
                }  
            }

            tranxy(&nx, &ny, n);
            P_f[ny][nx].person=1;
            check_win_person(P_f,mark);
            if (score(mark)==win){
                res=n;
                break;reset(mark);
            }

            tranxy(&nx, &ny, n);
            P_f[ny][nx].computer=1;
            check_win_computer(P_f,mark);
            if (score(mark)==win){
                res=n;
                break;reset(mark);
            }
        }
    }

    return res;
}
//ai brain for breaking threat
int ai_pre_step_two(const struct place P[3][3], int *mark){
    struct place P_f[3][3];
    int nx, ny, res=nothing, *rgs, m=0;
    rgs=(int *)malloc(number*sizeof(int));
    rgs[0] = 0;

    for(int n=1 ; n<=9 ; n++ ){

        if(check_exist(P,n)){
            for(int x = 0; x < 3; x++) { 
                for(int y = 0; y < 3; y++) {
                    P_f[y][x].all=P[y][x].all;  
                    P_f[y][x].computer = P[y][x].computer;
                    P_f[y][x].person = P[y][x].person;   
                }  
            }

            tranxy(&nx,&ny,n);
            P_f[ny][nx].person=1;
            check_win_person(P_f,mark);
            if (score(mark)==threat){
                P_f[ny][nx].all=nothing;
                for(int i=1 ; i<=9 ; i++ ){
                    if(check_exist(P_f,i)){
                        tranxy(&nx,&ny,i);
                        P_f[ny][nx].person=-1;
                        check_win_person(P_f,mark);
                        if (score(mark)!=threat && i!=n){
                            rgs[m]=i;
                            m++;
                        }
                        reset(mark);
                        P_f[ny][nx].person=0;
                    }
                }

                if (rgs[0]==0){
                    rgs[0]=n;
                }
            }
            reset(mark);

        }
    }

    int L=0;

    while(rgs[L]!=0){
        L++;
    }
    
    L=rand()%L;
    res=rgs[L];
    free(rgs);
    rgs=NULL;
    return res;
}
//ai brain for winning
int ai_pre_step_three(const struct place P[3][3],  int *mark){
    struct place P_f[3][3];
    int nx, ny, res=0;
    int *posb, L=0;
    posb=(int *)malloc(number*sizeof(int));
    for(int x = 0; x < 3; x++) { 
        for(int y = 0; y < 3; y++) {
            P_f[y][x].computer = P[y][x].computer - P[y][x].person;   
        }  
    }

    for(int n=1; n<=9; n++){
        if(check_exist(P,n)){
            tranxy(&nx,&ny,n);
            P_f[ny][nx].computer +=1;
            check_win_computer(P_f,mark);
            for (int i=0 ; i<8 ; i++){
                if (mark[i]==-2){
                    posb[L]=n;
                    L++;
                }
            }
            reset(mark);
        }
    }

    L=rand()%L;
    res=posb[L];
    free(posb);
    posb=NULL;
    return res;
}
//Decide who the final winner is
int WHO_win(const struct place P[3][3],int *mark){
    int res=draw;
    check_win_computer(P,mark);
    if (score(mark)==win){
        res=loose;
    }else{
        check_win_person(P,mark);
        if (score(mark)==win){
            res=win;
        }
    }
    return res;
}
//to decidde the first step
int r(void){
    int r;
    r=rand()%4;
    if(r==0){
        r=7;
    }else if (r==2){
        r=9;
    }
    return r;
}

int main (void){
    srand(time(0));

    struct place P[3][3];
    int n=0, i=1, nx, ny;
    int mark[8],mark_judge[8];
        
    fth_data(P);
    re_print(P);
    reset(mark_judge);

    while (score(mark_judge)!=win && i<=9){
        reset(mark_judge);reset(mark);
        if (i%2 == 1){
            scanf("%d",&n);
            if (n>=1 && n<=9){
               if (check_exist(P,n)){
                    tranxy(&nx,&ny,n);
                    P[ny][nx].person=1;
                    P[ny][nx].all=X;
                    re_print(P);
                }else {
                    printf("\nrepetition");
                    i--;
                } 
            }else{
                printf("invalid");
                i--;
            }
            check_win_person(P,mark_judge);
        }else if (i==2){
            if (P[1][1].all==X){
                n=r();
            }else{
                n=5;
            }
            tranxy(&nx,&ny,n);
            P[ny][nx].computer=1;
            P[ny][nx].all=O;
            re_print(P);
        }else{
            if(ai_pre_step_one(P, mark)!=nothing){
                n=ai_pre_step_one(P, mark);
                printf("\n采用1\n");
            }else if (ai_pre_step_two(P, mark)!=nothing){
                n=ai_pre_step_two(P, mark);
                printf("\n采用2\n");
            }else{
                n=ai_pre_step_three(P, mark);
                printf("\n采用3\n");
            }

            tranxy(&nx,&ny,n);
            P[ny][nx].computer=1;
            P[ny][nx].all=O;
            re_print(P);
            check_win_computer(P,mark_judge);
        }
        i++;
    }

    switch (WHO_win(P,mark_judge))
    {
    case win :
        printf("YOU WIN !\n");
        break;
    case draw :
        printf("MAKE A DRAW !\n");
        break;
    case loose :
        printf("YOU LOOSE !\n");
        break;
    }

    printf("END\n");
    
    char s;
    printf("I am Poter,the programmer of this game.\nThank you for your try!\n");
    printf("please input OK");
    scanf("%s",&s);
}
