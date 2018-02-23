# letanke
Robot 2.0

package ITC;
import robocode.*;
import java.awt.Color;

public class LeEdgyRazor extends Robot
{
	double direction = 45;
	double count = 5;
	double distance = 0;
	double previousBearing = 0;
	double X = 0;
	double Y = 0;
	double dodgeMove = 60;
	double bearing = 0;
	double enemyVelocity = 0;
	double myEnergy = 0;
	double changeInEnergy = 0;
	double previousEnergy = 0;
	double offBy = 0;
	double dodgeCounter = 0;
	double borderHeight = 0;
	double borderWidth = 0;
	double firePower = 0;
	boolean directionLeft = true;
	boolean directionRight = true;
	boolean dodge = false;
	int shotCounter = 0;


	public void run() {
		setAllColors(Color.ORANGE); 								
		borderHeight = getBattleFieldHeight();  					 // Trying to find my place in this crazy world
   		borderWidth = getBattleFieldWidth();
		borderWidth = borderWidth/2;								 // Centre
		borderHeight = borderHeight/2;
		while(true) {
	if (count < 5){
			turnRadarLeft(direction);
			turnRadarLeft(-direction);
			count++;
			}
		else {
				turnRadarRight(360);
			}
		
		}
	}
	
	public void onScannedRobot(ScannedRobotEvent e) {
		
			
		if (!e.isSentryRobot()){					  // Don't want to take on the beast
			count = 0;
			distance = e.getDistance();						// Getting the distance
			bearing = e.getBearing();							// Where the enemy is relative to me
			enemyVelocity = e.getVelocity();
			myEnergy = getEnergy();
			

			// PowerChanger Code
			if (myEnergy > 30){
				firePower = 2;
			}
			else if (myEnergy >10 && myEnergy < 30){
				firePower = 1;
			}
			else {
				firePower = 0.1;
				}
				
		
			//-----------------------------------------Aiming--------------------------------------------
			if (e.getBearing() > 3 || e.getBearing() < -3){
				turnRight(bearing + offBy); 
			}  
		
	
			//----------------------------------------Shooting-------------------------------------------
	   		if(getGunHeat() == 0.0){			
				if (distance < 200){
			    if(shotCounter == 1)
				{
					fire(2);
					shotCounter++;
				}
				else
				{
					fire(3);
					shotCounter++;
				}
				}
			else if (distance > 200){						// If he is far away, dodge and try and predict-ish
			
				// ------------------------------------Deciding the direction to prediction shoot--------------------------------------------
				if (previousBearing > bearing+2){							// My attempt at shot predicting, needs work
					directionLeft = true;
				}
				if (previousBearing < bearing-2){
					directionRight = true;
				}
				

					// ------------------------------------Prediction shooting--------------------------------------------
				if (directionRight == true && enemyVelocity != 0){
					turnGunRight(8);							// prediciton shot, not working so taken out
					fire(firePower);
						shotCounter++;
					turnGunLeft(8);					
					bearing = e.getBearing();				// setting new bearing??
				}
				
				if (directionLeft == true && enemyVelocity != 0){
					turnGunLeft(8);
					fire(firePower);
						shotCounter++;
					turnGunRight(8);;
					bearing = e.getBearing();
				}
				else{
					fire(firePower);  								 // No point in being shy
						shotCounter++;
				}
				
				if(dodge == false){									// Keep the gun to bearing ratio
					setAdjustGunForRobotTurn(true);							// if the enemy is afraid of you up close, enter parry state
					turnLeft(90);
					setAdjustGunForRobotTurn(false);
					offBy = -90;	
					dodge = true;
				}
			}
	
				// Bound check													
			if(dodge == false && distance > 80 && (getY() < (borderHeight + 80)) && (getY() > (borderHeight - 80))&& (getX() < (borderWidth + 80))&& (getX() > (borderWidth - 80))){
					ahead(30);
			}
			else if(dodge == false && distance > 60 && (getY() < (borderHeight + 80)) && (getY() > (borderHeight - 80))&& (getX() < (borderWidth + 80))&& (getX() > (borderWidth - 80))){
					ahead(10);
			}
			

			// ------------------------------------------------Dodging-----------------------------------------------  
			changeInEnergy = previousEnergy - e.getEnergy();
			if (changeInEnergy > 0 ){
				if (dodge == true)
				{
					ahead(dodgeMove);					// Dodge back and forth if their energy has changed
				if( (getY() > (borderHeight + 50)) || (getY() < (borderHeight - 50)) || (getX() > (borderWidth + 50))|| (getX() < (borderWidth - 50))){
							dodgeMove = -dodgeMove;
							dodgeCounter = 0;
							}
				}	
			}
			//
			bearing = e.getBearing();
			count = 0;							// In case we lose them, we never do
			previousBearing = bearing;			// Settings for the next loop
			previousEnergy = e.getEnergy();
			directionLeft = false;
			directionRight = false;
		}
	
		
		}
	}

	public void onHitByBullet(HitByBulletEvent e) {
	
		
	}

	public void onHitRobot(HitByBulletEvent e) {
		back(10);								// Just in case they get rowdy
	}
	

}

