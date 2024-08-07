this challenge is a commandline game and the main goal is to win  
  
the challenge files include  
	-player client  
	-bot source code  
	-game source code  
	-game server source code  
	-docker file for the deployement of the server  
  
  
game :  
-----  
 👾                                        
  
  
  
   🧱  
  
🚀                                         
-----  
  
  
we are the rocket and the main goal is to shoot the bot and win   
problems:  
	-the first 10 seconds of the game no shooting can be done  
	-the only move the bot does is to get behind the box in the center nothing more  
	-bot will get behind cover before the 10 second timer ends  
  
so how did i approach this?  
by reading the players client and server i understood that the client sends the messages in this format:  
f"{PLAYER_ID}|{action}|{option_for_action};"  
   
maybe we might wanna look into how players are handled more closely:  

```go
player id generation:  
this is happening at server side in the GenerateId() function:  
func GenerateId() int {   
	return rand.Intn(100)  
}  
```  
  
note that the range of player ids are 0-100 totally bruteforcable  
  
  
client message validation:  
receiving client input and validation is happenning in the handleConnection() function  
so lets breakdown the logic of the function  
1_read from the socket untill ';' is seen  
  
```go  
for {  
	tempN, err := conn.Read(tempBuffer)  
	if tempN == 0 || err != nil {  
		return  
	}  
	if tempBuffer[0] == ';' {  
		break  
	}  
	buffer = append(buffer, tempBuffer[0])  
	n += 1  
}  
```
  
2_check if player's socket match any of the sockets in the players list  

```go
playerExists := false  
for _, player := range g.Players {  
	if player.Conn == conn {  
		playerExists = true  
		break  
	}  
}  
``` 

3_call the function for the requested action  
	-if player doesn't exist only register action works  
  
4_go back to 1  
  
  
solution:  
  
so what if we place an id that isnt ours in client message format?  
	-remember an action is accepted if the player exists not necessarily matches the player id in the payload  
	-message format: reminder (f"{PLAYER_ID}|{action}|{option_for_action};")  
 
we can modify our client to send "move to left" action as all possible player ids  

```python3
for i in range (101):  
	socket.send(f"{i}|M|0;".encode())  
	print(f"sent with player id: {i}")   
```
   
this causes the bot to move to the left and out of the cover  
TFCCTF{this_might_have_a_race_condition}  


