# ESPeasy-rules
on System#Boot do
 GPIO,2,1    //led ki
 event,setup
endon

on setup do
 TaskValueSet 1,1,34  //min víz
 TaskValueSet 1,2,89  //over, biztonsági hőfok, amin el kezd keringetni
 TaskValueSet 2,1,350 //keringetési idő sec
 TaskValueSet 2,2,500  //keringetési idő szünet
 TaskValueSet 2,3,30  //termo figyel
 TaskValueSet 2,4,1200  //szivattyú védelem, ennyi kör után
 Let,2,2    //alap watch 
 Let,3,0   //protect
 TimerSet,2,8  //Start 
endon

on Rules#Timer=1 do  //pause
 Publish %sysname%/status, '{"Heating":"off","Pause":"on","Watch":"off"}'
 oled,7,3,Pause
 gpio,15,0
 TaskValueSet 4,1,0
 Let,1,[dtime#keringp]
 Let,2,2
 TimerSet,3,1
endon

on Rules#Timer=2 do  //watch
 if [owt#temp1]>[dtemp#tmax]
 event,hover
 elseif [termo#state]=1
 event,hon
 else
 event,hoff
 endif
endon

On  Rules#Timer=3 do  //timer
 if [VAR#1]<2
 Let,1,0
 TimerSet,3,0
 TimerSet,[VAR#2],1
 else
 Let,1,[VAR#1]-2
 TaskValueSet 4,2,[VAR#1]
 TimerSet,3,2
 endif
endon

on termo#state=0 do
 if [owt#temp1]<[dtemp#tmax]
 TimerSet,1,3
 else
 TimerSet,2,1
 endif
endon

on hon do
 if [owt#temp1]>[dtemp#tmin] and [ha#ha]=0
 Publish %sysname%/status, '{"Heating":"on","Pause":"off","Watch":"off"}'
 oled,7,3,Heating!
 gpio,15,1
 TaskValueSet 4,1,1
 TaskValueSet 4,3,0
 Let,1,([dtime#kering]-([owt#temp1]*2))
 Let,2,1
 Let,3,0
 TimerSet,3,1
 else
 event,hoff
 endif
endon

on hover do
 oled,7,3,OVERHEAT
 gpio,15,1
 TaskValueSet 4,1,1
 Let,1,600
 Let,2,2
 Let,3,0
 TimerSet,3,1
endon

on hoff do
 gpio,15,0
 TaskValueSet 4,1,0
 Let,3,[VAR#3]+1
 TaskValueSet 4,3,[VAR#3]
 if [var#3]=[dtime#protect] 
 Publish %sysname%/status, '{"Heating":"on","Pause":"off","Watch":"off"}'
 oled,7,3, Protect
 gpio,15,1
 TaskValueSet 4,3,0
 TaskValueSet 4,1,1
 Let,1,60
 Let,2,2
 Let,3,0
 Timerset 3,1
 elseif [ha#ha]=1
 Publish %sysname%/status, '{"Heating":"off","Pause":"off","Watch":"off"}'
 oled,7,3, Ablakok
 TimerSet,2,[dtime#watch]
 endif
 else
 Publish %sysname%/status, '{"Heating":"off","Pause":"off","Watch":"on"}'
 oled,7,3, Watch
 TimerSet,2,[dtime#watch]
 endif
endon
