Firsly you need to have Eclipse IDE for Java Developers and also JRE 1.7
Set it up with the Eclipse IDE , and then use this code and have fun exploring Leap :


import com.leapmotion.leap.*;
import com.leapmotion.leap.Gesture.State;

import java.io.IOException;

class LeapListener extends Listener {
	public void onInit(Controller controller) {
		System.out.println("Initialized");
	}
	
	public void onConnect(Controller controller) {
		System.out.println("Connected to leap!");
		controller.enableGesture(Gesture.Type.TYPE_SWIPE);
		controller.enableGesture(Gesture.Type.TYPE_CIRCLE);
		controller.enableGesture(Gesture.Type.TYPE_SCREEN_TAP);
		controller.enableGesture(Gesture.Type.TYPE_KEY_TAP);
		
	}
	
	public void onDisconnect(Controller controller) {
		System.out.println("Disconnected leap!");
	}
	
	public void onExit(Controller controller) {
		System.out.println("Exited!");
	}
	
	public void onFrame(Controller controller) {
		Frame frame = controller.frame();
		 System.out.println("Frame id: " + frame.id()
				+ ", Timestamp: " + frame.timestamp()
				+ ", Hands:" + frame.hands().count()
				+ ", Fingers:" + frame.fingers().count()
				+ ", Tools" + frame.tools().count()
				+ ", Gestures" + frame.gestures().count()); 
		
		 for (Hand hand : frame.hands()) {
			String handType = hand.isLeft() ? "Left Hand" : "Right Hand";
			System.out.println(handType + " " + ", id:" + hand.id()  
					+ ", Palm Position:" + hand.palmPosition());
			
			Vector normal = hand.palmNormal();
			Vector direction = hand.direction();
			
			System.out.println("Pitch: " + Math.toDegrees(direction.pitch())
					+ " Roll : " + Math.toDegrees(normal.roll())
					+ " Yaw : " + Math.toDegrees(direction.yaw()));
		} 
		
		 for (Finger finger : frame.fingers()) {
			System.out.println("Finger Type: " + finger.type()
					+ " ID : " + finger.id()
					+ "Finger Length (mm) :" + finger.length()
					+ "Finger Width (mm) : " + finger.width());
			
			for (Bone.Type boneType : Bone.Type.values()) {
				Bone bone = finger.bone(boneType);
				System.out.println("Bone Type : " + bone.type()
						+ " Start : " + bone.prevJoint()
						+ "End : " + bone.nextJoint()
						+ "Bone Direction :" + bone.direction());
			}
		} 
		
		 for (Tool tool : frame.tools()) {
			System.out.println("Tool ID : " + tool.id()
					+ " Tip Position : " + tool.tipPosition()
					+ "Direction" + tool.direction()
					+ "Width (mm) : " + tool.width()
					+ "Touch Distance" + tool.touchDistance() );
		} 
		
		GestureList gestures = frame.gestures();
		for (int i=0; i < gestures.count(); i++) {
			Gesture gesture = gestures.get(i);
			
			switch (gesture.type()) {
			case TYPE_CIRCLE:
				CircleGesture circle = new CircleGesture(gesture);
				
				String clockwiseness;
				if (circle.pointable().direction().angleTo(circle.normal())  <= Math.PI/4) {
					
					clockwiseness = "clockwise";
					
					
				}
				else {
					clockwiseness = "counter-clockwise";
				}
				
				double sweptAngle = 0;
				if (circle.state() != State.STATE_START) {
					CircleGesture previous = new CircleGesture(controller.frame(1).gesture(circle.id()));
					sweptAngle = (circle.progress() - previous.progress()) * 2 * Math.PI;
					
				}
					
				System.out.println("Circle ID : " + circle.id()
						 + "State: " + circle.state()
						 + "Progress: " + circle.progress()
						 + "Radius" + circle.radius()
						 + " Angle : " + Math.toDegrees(sweptAngle)
						 + "" + clockwiseness);
				break;
				
			}
		}
		
		
	}
}

public class LeapController {

	public static void main(String[] args) {
		LeapListener listener = new LeapListener();
		Controller controller = new Controller();
		
		controller.addListener(listener);
		
		System.out.println("Press enter to quit");
		
		try {
			System.in.read();
		} catch (IOException e){
			e.printStackTrace();
		}

		controller.removeListener(listener);
	}

}
