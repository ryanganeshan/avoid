#include <libplayerc++/playerc++.h>
#include <iostream>

    using namespace PlayerCc;

    std::string  gHostname(PlayerCc::PLAYER_HOSTNAME);
    // NOTE !!! PLAYER_PORTNUM is the port number and has to be unique for each student,
   // and be set in the plaer server with -p
    uint32_t        gPort(PlayerCc::PLAYER_PORTNUM); // Replace this with your port number !!


int main(int argc, char **argv)
{

  // we throw exceptions on creation if we fail
  try {

    PlayerClient robot(gHostname, gPort); // Conect to server
    Position2dProxy pp(&robot, 0);   // Get a motor control device (index is 0)
    RangerProxy ranger(&robot, 0); // Could also be 1 for the laser
    // ONE READ IS NEEDED AT THE BEGINNING TO CONNECT
    robot.Read();
    robot.Read();
    robot.Read();
    robot.Read();
    robot.Read();
    robot.Read();
    robot.Read();
    std::cout << robot << std::endl;
    ranger.RequestGeom();
    ranger.RequestConfigure();
    // Now you can get information for example
    int num = ranger.GetRangeCount(); // Gives you the number of range readings
    double min_ang=ranger.GetMinAngle (); // Gives the minimum  angle
    double val = ranger[3]; // gives you the reading with index 3
    //obviously the angle corespoinding to reading 3 is 
    double abgle = ranger.GetMinAngle () + 3* ranger.GetRangeRes();
    double range0 = ranger.GetRange(0);
    double range1 = ranger.GetRange(1);
    double range2 = ranger.GetRange(2);
    double range3 = ranger.GetRange(3);
    double range4 = ranger.GetRange(4);
    double range5 = ranger.GetRange(5);

    pp.SetMotorEnable (true); // Turn on Motors
    pp.SetSpeed(1,0);


    // go into  a loop
    for(;;){
      double newspeed = .5 ;
      double newturnrate = 0;

      // this blocks until new data comes; 10Hz by default
      robot.Read();
    
      range0 = ranger.GetRange(0);
      range1 = ranger.GetRange(1);
      range2 = ranger.GetRange(2);
      range3 = ranger.GetRange(3);
      range4 = ranger.GetRange(4);
      range5 = ranger.GetRange(5);

      std::cout << "num: " << num << std::endl;
      std::cout << "min_ang: " << min_ang << std::endl;
      std::cout << "val: " << val << std::endl;
      std::cout << "abgle: " << abgle << std::endl;
      std::cout << "range[0]: " << range0 << std::endl;
      std::cout << "range[1]: " << range1 << std::endl;
      std::cout << "range[2]: " << range2 << std::endl;
      std::cout << "range[3]: " << range3 << std::endl;
      std::cout << "range[4]: " << range4 << std::endl;
      std::cout << "range[5]: " << range5 << std::endl;
      
      if((range0 < 1.0) || (range4 < 1.0) || (range3 < 1.0) || (range2 < 1.0) || (range1 < 1.0)) {
          pp.SetSpeed(0,0);
          if(range1 > range2) {
              while((range0 < 1.0) && (range1 > 0.5) || ((range3 < 0.75) || (range4 < 0.75))) {
                  pp.SetSpeed(0, -0.5);
                  range0 = ranger.GetRange(0);
                  range2 = ranger.GetRange(2);
                  range3 = ranger.GetRange(3);
                  range4 = ranger.GetRange(4);
                  robot.Read();
              }
              pp.SetSpeed(1,0);
          }
          if(range2 > range1) {
              while((range0 < 1.0) && (range2 > 0.5) || ((range3 < 0.75) || (range4 < 0.75))) {
                  pp.SetSpeed(0, 0.5);
                  range0 = ranger.GetRange(0);
                  range1 = ranger.GetRange(1);
                  range3 = ranger.GetRange(3);
                  range4 = ranger.GetRange(4);
                  robot.Read();
              }
              pp.SetSpeed(1,0);
          }
      }

      // write commands to robot
      pp.SetSpeed(newspeed, newturnrate);

    }
  }
  catch (PlayerCc::PlayerError & e) {
    std::cerr << e << std::endl;
    return -1;
  }
}
