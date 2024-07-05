# Vehicle-Cut-In-Detection
package platoon_1_6;


 /*
 INPUTS:
0: Host vehicle x position
1: Host vehicle y position
1: Host vehicle speed
2: Cut in braking status

 OUTPUTS:
0: Cut-in modifier
1: Distance to cut-in vehicle
2: Relative speed of cut-in vehicle
*/
public class CutInDetector extends FunctionBlock {

private Platoon_Vehicle hostVehicle;

private static final int SAMPLE_RATE = 20;
 // CID range, or the greatest distance at which the CID operates
 private static final double TOO_FAR_AWAY = 100;
 private static final double laneWidth = 4;
 private static final double vehicleWidth = 3;
 private static final double vehicleLength = 18; //this is x_r_0

public static final int STATUS_CURRENTLY_CUT_IN_BRAKING = 1;
public static final int STATUS_NOT_CUT_IN_BRAKING = 0;

public static final int MODIFIER_NO_CHANGE = 0;
public static final int MODIFIER_START_CUT_IN_BRAKING = 2;
public static final int MODIFIER_STOP_CUT_IN_BRAKING = 1;

private double cutInVehicleX;
private double cutInVehicleSPD;

private double hostX;
private double hostY;

public CutInDetector(Platoon_Vehicle hostV) {
hostVehicle = hostV;
}

@Override
public void run(long ticks) {
 if ((ticks % SAMPLE_RATE) == 0) {
 double[] inputs = readReg();
 double[] outputs = buildOutputArray();
 hostX = inputs[0];
 hostY = inputs[1];
 double hostSpeed = inputs[2];
 double status = inputs[3];
 double mostUrgentDistance = 100000; // just a starting value
 double mostUrgentSpeed = 0; // just a starting value

 boolean actionNeeded = false;
 
 for (Object obj : hostVehicle.getObjectList()) {

double extVehicleX 
 = ((Platoon_Vehicle) obj).register
[Platoon_Vehicle.REGISTER_HOST_POSITION_X];

if (obj instanceof Platoon_Vehicle
 && extVehicleX - hostX - vehicleLength < 
TOO_FAR_AWAY) {

double extVehicleSpeed 
 = ((Platoon_Vehicle) obj).register
[Platoon_Vehicle.REGISTER_VEHICLE_SPEED_REAL];
 double relativeSpeed = extVehicleSpeed - hostSpeed;
 double relativeDist = extVehicleX - hostX;

 if (relativeDist < vehicleLength) {
 System.err.println("crash at x = " + hostX 
 + " m at a speed of " + relativeSpeed + " km/
h");
 }

 if (changingIntoOurLane((Platoon_Vehicle) obj)) {

 if (relativeSpeed < -0.01) {
actionNeeded = true;

 if (relativeDist < mostUrgentDistance) {
 mostUrgentDistance = relativeDist;
 mostUrgentSpeed = relativeSpeed;
 cutInVehicleX = extVehicleX;
 cutInVehicleSPD = extVehicleSpeed;
 }
 }
 } else if (fullyInOurLane((Platoon_Vehicle) obj)) {

 if (relativeSpeed < 0) {
 actionNeeded = true;

 if (relativeDist < mostUrgentDistance) {
 mostUrgentDistance = relativeDist;
 mostUrgentSpeed = relativeSpeed;
 cutInVehicleX = extVehicleX;
 cutInVehicleSPD = extVehicleSpeed;
 }
 }
 }
 }
 }
 if (actionNeeded) {
 if (status == STATUS_NOT_CUT_IN_BRAKING) {
 outputs[0] = MODIFIER_START_CUT_IN_BRAKING;
 outputs[1] = mostUrgentDistance;
 outputs[2] = mostUrgentSpeed;
 } else if (status == STATUS_CURRENTLY_CUT_IN_BRAKING) {
 outputs[0] = MODIFIER_NO_CHANGE;
 outputs[1] = mostUrgentDistance;
 outputs[2] = mostUrgentSpeed;
 }
 } else if (status == STATUS_CURRENTLY_CUT_IN_BRAKING) {
 outputs[0] = MODIFIER_STOP_CUT_IN_BRAKING;
 outputs[1] = Platoon_Vehicle.initVal;
 outputs[2] = Platoon_Vehicle.initVal;
 double cutInVehicleX = ((Platoon_Vehicle) hostVehicle
 .getObjectList().get(0))
 .register
[Platoon_Vehicle.REGISTER_HOST_POSITION_X];
 double hostX = hostVehicle
 .register[Platoon_Vehicle.REGISTER_HOST_POSITION_X]

 }
 writeReg(outputs);
 }
 }

 @Override
 public void reset(double initialSPDms) {

 }

 // Investigates whether a vehicle is changing into host's lane. assumes 
that
 // host is travelling along the x-axis in positive direction.
 private boolean changingIntoOurLane(Platoon_Vehicle vehicle) {
 double leftLaneEdge = 0 + laneWidth / 2;
 double rightLaneEdge = 0 - laneWidth / 2;
 double vehicleY = vehicle.register
[Platoon_Vehicle.REGISTER_HOST_POSITION_Y];
 double vehicleLeftEdge = vehicleY + vehicleWidth / 2;
 double vehicleRightEdge = vehicleY - vehicleWidth / 2;
 return ((vehicleLeftEdge > leftLaneEdge
 && vehicleRightEdge < leftLaneEdge)
 || (vehicleLeftEdge > rightLaneEdge
 && vehicleRightEdge < rightLaneEdge));
 }

 // Investigates whether a vehicle is fully in host's lane. assumes that
 // host is travelling along the x-axis in positive direction. 
 private boolean fullyInOurLane(Platoon_Vehicle vehicle) {
 double leftLaneEdge = hostY + laneWidth / 2;
 double rightLaneEdge = hostY - laneWidth / 2;
 double vehicleY = vehicle.register
[Platoon_Vehicle.REGISTER_HOST_POSITION_Y];
 double vehicleLeftEdge = vehicleY + vehicleWidth / 2;
 double vehicleRightEdge = vehicleY - vehicleWidth / 2;
 return (vehicleLeftEdge < leftLaneEdge
 && vehicleRightEdge > rightLaneEdge);
 }

}//CutInDetector
