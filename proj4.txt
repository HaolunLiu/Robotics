#!/usr/bin/env pybricks-micropython
from pybricks.hubs import EV3Brick
from pybricks.ev3devices import (Motor, TouchSensor, ColorSensor,
                                 InfraredSensor, UltrasonicSensor, GyroSensor)
from pybricks.parameters import Port, Stop, Direction, Button, Color
from pybricks.tools import wait, StopWatch, DataLog
from pybricks.robotics import DriveBase
from pybricks.media.ev3dev import SoundFile, ImageFile
import array

# This program requires LEGO EV3 MicroPython v2.0 or higher.
# Click "Open user guide" on the EV3 extension tab for more information.


# Create your objects here.
ev3 = EV3Brick()
dis_sensor= UltrasonicSensor(Port.S1)
left_motor = Motor(Port.B)
right_motor = Motor(Port.C)
spin = Motor(Port.A, Direction.CLOCKWISE)
robot = DriveBase(left_motor, right_motor, wheel_diameter=55, axle_track=111)


ScreenHeight = 127
ScreenWidth = 177
RobotDirection = 0#0=North, 1=East, 2=South, 3=West
StartPosRow = 3
StartPosCol = 3
CurrentPosRow = StartPosRow
CurrentPosCol = StartPosCol
TargetPosRow = 0
TargetPosCol = 5
rightangle = 78
reverse = 155

#walls. 1 is wall, 2 is been through
class cell():
    def __init__(self):
        self.NorthWall = 0
        self.SouthWall = 0
        self.EastWall = 0
        self.WestWall = 0
Grid = [[cell() for i in range(6)] for j in range(4)]

def GridInit():
    for i in range(4):
		for j in range(6):
			Grid[i][j].NorthWall=1
			Grid[i][j].EastWall=1
			Grid[i][j].WestWall=1
			Grid[i][j].SouthWall=1

def WallGen():
	i = 0
	j = 0
	for i in range(4):
		Grid[i][0].WestWall=1
		Grid[i][5].EastWall=1
	

	for j in range(6):
		Grid[0][j].SouthWall=1
		Grid[3][j].NorthWall=1

	Grid[0][0].NorthWall =1
	Grid[1][0].SouthWall =1
	Grid[0][1].NorthWall =1 
	Grid[1][1].SouthWall =1
	Grid[0][3].EastWall  =1  
	Grid[0][4].WestWall  =1
	Grid[1][2].EastWall  =1  
	Grid[1][3].WestWall  =1
	Grid[1][3].EastWall  =1  
	Grid[1][4].WestWall  =1
	Grid[1][4].EastWall  =1  
	Grid[1][5].WestWall  =1
	Grid[1][5].NorthWall =1  
	Grid[2][5].SouthWall =1
	Grid[3][0].EastWall  =1  
	Grid[3][1].WestWall  =1
	Grid[3][4].SouthWall =1  
	Grid[2][4].NorthWall =1

	for j in range(2, 4):
		Grid[2][j].NorthWall=1
		Grid[2][j].SouthWall=1
		Grid[3][j].SouthWall=1
		Grid[1][j].NorthWall=1
# #three walls are 1 or forced to move back, mark that block to solid
#the west wall for this block is the east wall for the block to the left 
#treat every block individually
#sensor to the right hug right wall button to detect in front
#code:
#start by 180 degree turning right cuz hugging right wall
#change bot position based on movement made (currentPosrow,col)
#go drive until hit wall or sensor detect wall; record wall values
#when button pressed means hit wall, stop and reverse to middle of current block
#everytime button pressed and return, turn left and continue hugging right
#when crossing a wall already been through, turn right
#if turned back the block should be neglected from point on.
#when in the target block, stop.
#return algo:
#return through all the walls where value is been through so 3
#turn 180 to face the way came from, next block
#find which wall is value 3 and turn towards
#continue next block
#when do go through a wall, the wall behind becomes 2
#when wall gets crossed twice, it becomes wall again
#first time cross wall, value becomes 0 (go throughable), cross again becomes 1 (wall) if wall value is 0 whencrossing change to 1
def cellCh(CurrentPosCol, CurrentPosRow, Grid, RobotDirection):
	if RobotDirection == 0:
		if Grid[CurrentPosRow][CurrentPosCol].NorthWall == 0:
			Grid[CurrentPosRow][CurrentPosCol].NorthWall = 1
			CurrentPosRow -= 1
			Grid[CurrentPosRow][CurrentPosCol].SouthWall = 1
			print("north")
			return CurrentPosCol, CurrentPosRow, Grid
		else:
			Grid[CurrentPosRow][CurrentPosCol].NorthWall = 0
			CurrentPosRow -= 1
			Grid[CurrentPosRow][CurrentPosCol].SouthWall = 0
			print("north")
			return CurrentPosCol, CurrentPosRow, Grid
	elif RobotDirection == 1:
		if Grid[CurrentPosRow][CurrentPosCol].EastWall ==0:
			Grid[CurrentPosRow][CurrentPosCol].EastWall = 1
			CurrentPosCol += 1
			Grid[CurrentPosRow][CurrentPosCol].WestWall = 1
			print("east")
			return CurrentPosCol, CurrentPosRow, Grid
		else:
			Grid[CurrentPosRow][CurrentPosCol].EastWall = 0
			CurrentPosCol += 1
			Grid[CurrentPosRow][CurrentPosCol].WestWall = 0
			print("east,,")
			return CurrentPosCol, CurrentPosRow, Grid
	elif RobotDirection == 2:
		if Grid[CurrentPosRow][CurrentPosCol].SouthWall ==0:
			Grid[CurrentPosRow][CurrentPosCol].SouthWall = 1
			CurrentPosRow += 1
			Grid[CurrentPosRow][CurrentPosCol].NorthWall = 1
			print("south!!!")
			return CurrentPosCol, CurrentPosRow, Grid
		else:
			Grid[CurrentPosRow][CurrentPosCol].SouthWall = 0
			CurrentPosRow += 1
			Grid[CurrentPosRow][CurrentPosCol].NorthWall = 0
			print("south")
			return CurrentPosCol, CurrentPosRow, Grid
	elif RobotDirection == 3:
		if Grid[CurrentPosRow][CurrentPosCol].WestWall ==0:
			Grid[CurrentPosRow][CurrentPosCol].WestWall = 1
			CurrentPosCol -= 1
			Grid[CurrentPosRow][CurrentPosCol].EastWall = 1
			print("west")
			return CurrentPosCol, CurrentPosRow, Grid
		else:
			Grid[CurrentPosRow][CurrentPosCol].WestWall = 0
			CurrentPosCol -= 1
			Grid[CurrentPosRow][CurrentPosCol].EastWall = 0
			print("west")
			return CurrentPosCol, CurrentPosRow, Grid
	
def wallisfront(CurrentPosCol, CurrentPosRow, Grid, RobotDirection):
 	if RobotDirection == 0:
 		Grid[CurrentPosRow][CurrentPosCol].NorthWall = 1
 		return Grid
 	elif RobotDirection == 1:
 		Grid[CurrentPosRow][CurrentPosCol].EastWall = 1
 		return Grid
 	elif RobotDirection == 2:
 		Grid[CurrentPosRow][CurrentPosCol].SouthWall = 1
 		return Grid
 	else:
 		Grid[CurrentPosRow][CurrentPosCol].WestWall = 1
 		return Grid
def wallisleft(CurrentPosCol, CurrentPosRow, Grid, RobotDirection):
 	if RobotDirection == 0:
 		Grid[CurrentPosRow][CurrentPosCol].WestWall = 1
 		return Grid
 	elif RobotDirection == 1:
 		Grid[CurrentPosRow][CurrentPosCol].NorthWall = 1
 		return Grid
 	elif RobotDirection == 2:
 		Grid[CurrentPosRow][CurrentPosCol].EastWall = 1
 		return Grid
 	else:
 		Grid[CurrentPosRow][CurrentPosCol].SouthWall = 1
 		return Grid

def wallisright(CurrentPosCol, CurrentPosRow, Grid, RobotDirection):
 	if RobotDirection == 0:
 		Grid[CurrentPosRow][CurrentPosCol].EastWall = 1
 		return Grid
 	elif RobotDirection == 1:
 		Grid[CurrentPosRow][CurrentPosCol].SouthWall = 1
 		return Grid
 	elif RobotDirection == 2:
 		Grid[CurrentPosRow][CurrentPosCol].WestWall = 1
 		return Grid
	else:
 		Grid[CurrentPosRow][CurrentPosCol].NorthWall = 1
 		return Grid
		
def adj():
	leftdis = 0
	rightdis = 0
	robot.stop(Stop.BRAKE)
	spin.reset_angle(0)
	spin.run_target(200, -95, Stop.BRAKE, True) #read left wall
	
	leftdis = dis_sensor.distance()
	spin.reset_angle(0)
	spin.run_target(200, 191, Stop.BRAKE, True) #read right wall
	
	rightdis = dis_sensor.distance()
	spin.reset_angle(0)
	spin.run_target(200, -95, Stop.BRAKE, True) #turn back front 
	#if no walls open return
	temp = rightdis - leftdis 
	temp1 = leftdis - rightdis
	if temp > 0 :
		if temp < 6:
			robot.turn(1)
		elif temp < 10:
			robot.turn(2)
		elif temp < 15:
			robot.turn(3)
		elif temp < 25:
			robot.turn(4)
		elif temp < 45:
			robot.turn(5)
		elif temp < 60:
			robot.turn(6)
		else:
			robot.turn(0)
	if temp1 > 0 :
		if temp1 < 6:
			robot.turn(-1)
		elif temp1 < 10: 
			robot.turn(-2)
		elif temp1 < 15:
			robot.turn(-3)
		elif temp1 < 25:
			robot.turn(-4)
		elif temp1 < 45:
			robot.turn(-5)
		elif temp1 < 60:
			robot.turn(-6)
		else: 
			robot.turn(0)
	

def Solver(RobotDirection: int, CurrentPosCol, CurrentPosRow, Grid):
	wallright = True
	wallleft = True
	wallfront = True
	robot.reset()
	robot.stop(Stop.BRAKE)
	if dis_sensor.distance() < 120: # wall infront
		while dis_sensor.distance() > 45:
			robot.drive(70, 0)
		robot.stop(Stop.BRAKE) 
		Grid = wallisfront(CurrentPosCol, CurrentPosRow, Grid, RobotDirection)#wall front
	else: # no wall  
		wallfront = False 
	spin.reset_angle(0)
	spin.run_target(200, -95, Stop.BRAKE, True) #read right wall
	wait(10)
	rightdis=dis_sensor.distance()
	ev3.screen.print(dis_sensor.distance())
	if dis_sensor.distance() < 140:
		Grid = wallisright(CurrentPosCol, CurrentPosRow, Grid, RobotDirection)
	else:
		wallright = False
	spin.reset_angle(0)
	spin.run_target(200, 191, Stop.BRAKE, True) #read left wall
	wait(10)
	leftdis = dis_sensor.distance()
	ev3.screen.print(dis_sensor.distance())
	if dis_sensor.distance() < 140: #120
		Grid = wallisleft(CurrentPosCol, CurrentPosRow, Grid, RobotDirection)
	else:
		wallleft = False
		#no wall there

		#nowall there
	spin.reset_angle(0)
	spin.run_target(200, -95, Stop.BRAKE, True) #turn back front 
	#if no walls open return

	if wallright == False: #if no right wall, turn right and break
		robot.reset()
		robot.turn(rightangle)
		if RobotDirection == 3:
			RobotDirection = 0
		else: 
			RobotDirection = RobotDirection + 1
		adj()
		robot.straight(228)
		CurrentPosCol, CurrentPosRow, Grid = cellCh(CurrentPosCol, CurrentPosRow, Grid, RobotDirection)
		return RobotDirection, CurrentPosCol, CurrentPosRow, Grid	
	elif wallfront == False:
		robot.reset()
		adj()
		robot.straight(228)
		CurrentPosCol, CurrentPosRow, Grid = cellCh(CurrentPosCol, CurrentPosRow, Grid, RobotDirection)	
		return RobotDirection, CurrentPosCol, CurrentPosRow, Grid			
	elif wallleft == False:                    #else only left wall open turn left
		robot.reset()
		robot.turn(-rightangle)
		if RobotDirection == 0:
			RobotDirection = 3
		else: 
			RobotDirection = RobotDirection - 1
		adj()
		robot.straight(228)
		CurrentPosCol, CurrentPosRow, Grid = cellCh(CurrentPosCol, CurrentPosRow, Grid, RobotDirection)
		return RobotDirection, CurrentPosCol, CurrentPosRow, Grid
	else:
		robot.reset()
		robot.turn(reverse)
		if RobotDirection == 0:
			RobotDirection = 2
		elif RobotDirection == 1:
			RobotDirection = 3
		else: 
			RobotDirection = RobotDirection - 2
		adj()


		robot.straight(228)
		CurrentPosCol, CurrentPosRow, Grid = cellCh(CurrentPosCol, CurrentPosRow, Grid, RobotDirection)	
		return RobotDirection, CurrentPosCol, CurrentPosRow, Grid
	return RobotDirection, CurrentPosCol, CurrentPosRow, Grid
						
def Oreturn(RobotDirection: int,CurrentPosCol, CurrentPosRow, Grid):
	if RobotDirection == 0:
		if Grid[CurrentPosRow][CurrentPosCol].NorthWall == 0:
			robot.stop(Stop.BRAKE)
		elif Grid[CurrentPosRow][CurrentPosCol].WestWall == 0:
			robot.reset()
			robot.turn(-rightangle)
			RobotDirection = 3
		elif Grid[CurrentPosRow][CurrentPosCol].EastWall == 0:
			robot.reset()
			robot.turn(rightangle)
			RobotDirection = 1
		else:
			robot.reset()
			robot.turn(reverse)
			RobotDirection = 2
	elif RobotDirection == 1:
		if Grid[CurrentPosRow][CurrentPosCol].EastWall == 0:
			robot.stop(Stop.BRAKE)
		elif Grid[CurrentPosRow][CurrentPosCol].NorthWall == 0:
			robot.reset()
			robot.turn(-rightangle)
			RobotDirection = 0		
		elif Grid[CurrentPosRow][CurrentPosCol].SouthWall == 0:
			robot.reset()
			robot.turn(rightangle)
			RobotDirection = 2
		else:
			robot.reset()
			robot.turn(reverse)
			RobotDirection = 3
	elif RobotDirection == 2:
		if Grid[CurrentPosRow][CurrentPosCol].SouthWall == 0:
			robot.stop(Stop.BRAKE)
		elif Grid[CurrentPosRow][CurrentPosCol].WestWall == 0:
			robot.reset()
			robot.turn(rightangle)
			RobotDirection = 3
		elif Grid[CurrentPosRow][CurrentPosCol].EastWall == 0:
			robot.reset()
			robot.turn(-rightangle)
			RobotDirection = 1
		else:
			robot.reset()
			robot.turn(reverse)
			RobotDirection = 0

	elif RobotDirection == 3:
		if Grid[CurrentPosRow][CurrentPosCol].WestWall == 0:
			robot.stop(Stop.BRAKE)
		elif Grid[CurrentPosRow][CurrentPosCol].NorthWall == 0:
			robot.reset()
			robot.turn(rightangle)
			RobotDirection = 0
		elif Grid[CurrentPosRow][CurrentPosCol].SouthWall == 0:
			robot.reset()
			robot.turn(-rightangle)
			RobotDirection = 2
		else:
			robot.reset()
			robot.turn(reverse)
			RobotDirection = 1

	adj()
	CurrentPosCol, CurrentPosRow, Grid = cellCh(CurrentPosCol, CurrentPosRow, Grid, RobotDirection)
	robot.straight(228)
	return RobotDirection, CurrentPosCol, CurrentPosRow, Grid

gameover = False
GridInit()
WallGen()
#robot.settings(200, 200, 100)
while True:
	if CurrentPosRow != TargetPosRow or CurrentPosCol != TargetPosCol:
		RobotDirection, CurrentPosCol, CurrentPosRow, Grid = Solver(RobotDirection, CurrentPosCol, CurrentPosRow, Grid)

	else:
		ev3.speaker.beep()
		robot.stop(Stop.BRAKE)
		if RobotDirection == 0:
			Grid[TargetPosRow][TargetPosCol].EastWall = 1
			Grid[TargetPosRow][TargetPosCol].NorthWall = 1
			Grid[TargetPosRow][TargetPosCol].WestWall = 1
		elif RobotDirection == 1:
			Grid[TargetPosRow][TargetPosCol].NorthWall = 1
			Grid[TargetPosRow][TargetPosCol].EastWall = 1
			Grid[TargetPosRow][TargetPosCol].SouthWall = 1
		elif RobotDirection == 2:
			Grid[TargetPosRow][TargetPosCol].WestWall = 1
			Grid[TargetPosRow][TargetPosCol].EastWall = 1
			Grid[TargetPosRow][TargetPosCol].SouthWall = 1
		else:
			Grid[TargetPosRow][TargetPosCol].NorthWall = 1
			Grid[TargetPosRow][TargetPosCol].WestWall = 1
			Grid[TargetPosRow][TargetPosCol].SouthWall = 1
		gameover = True

		while True:
			if CurrentPosRow != StartPosRow or CurrentPosCol != StartPosCol:
				ev3.screen.print(RobotDirection)
				ev3.screen.print(CurrentPosRow, CurrentPosCol)
				ev3.screen.print(Grid[CurrentPosRow][CurrentPosCol].NorthWall, Grid[CurrentPosRow][CurrentPosCol].EastWall, Grid[CurrentPosRow][CurrentPosCol].SouthWall, Grid[CurrentPosRow][CurrentPosCol].WestWall)
				RobotDirection, CurrentPosCol, CurrentPosRow, Grid = Oreturn(RobotDirection, CurrentPosCol, CurrentPosRow, Grid)
			else:
				break
		break
ev3.speaker.beep()
while gameover == True:
	ev3.screen.draw_text(5,0,"MAZE SOLVED !!")
 	wait(300)
 	ev3.screen.clear()
	wait(300)



