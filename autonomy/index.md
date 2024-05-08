# Pathplanner

## Basic AutoBuilder for Swerve drive
Here we have the basic code for autobuilder from our 2024 robot code located in the [DriveSubsystem.cpp](https://github.com/FRC4903/dev2024/blob/main/src/main/cpp/subsystems/DriveSubsystem.cpp?plain=1#L53) file
```cpp
    AutoBuilder::configureHolonomic(
      [this](){ return GetEstimatedPose(); },
      [this](frc::Pose2d pose){ ResetOdometry(pose); },
      [this](){ return getRobotRelativeSpeeds(); },
      [this](frc::ChassisSpeeds speeds){ Drive(speeds.vx,speeds.vy,speeds.omega,false); },
      HolonomicPathFollowerConfig(
          PIDConstants(5.0, 0.0, 0.0), 
          PIDConstants(5.0, 0.0, 0.0), 
          3.0_mps, 
          0.5_m, 
          ReplanningConfig()
      ),
      [this]()->bool{return this ->Fieldflip();},
      this
    );
```
Most of the paramaters are asking for a suppiler

### Robot Pose suppiler
The GetEmtimatedPose() function returns the pose2D from the m_odometry object the m_odometry object
the odometry object estimates the position of the robot on the field
(Hopefully odometry is explained in vehicle dynamics)
```cpp
frc::Pose2d DriveSubsystem::GetEstimatedPose() {
  return m_odometry.GetEstimatedPosition();
}
```

### Reset Odometry
ResetOdemery() is very similar to the pose suppiler all it does is call the reset odometry method on the m_odometry object

```cpp
void DriveSubsystem::ResetOdometry(frc::Pose2d pose) {
  m_odometry.ResetPosition(
      GetHeading(),
      {m_frontLeft.GetPosition(), m_frontRight.GetPosition(),
       m_rearLeft.GetPosition(), m_rearRight.GetPosition()},
      pose);
}
```

The parameters for the reset are the heading of the robot which is the current gyro rotation, the four swerve module states and, the pose we want to reset to


### Get Robot Relative Speeds
getRobotRelativeSpeeds() here we get the state of each swerve module and return the speed using the ToChassisSpeeds method on
kDriveKinematics which is a frc::SwerveDriveKinatics object (it does a lot of funny math for us)

```cpp
frc::ChassisSpeeds DriveSubsystem::getRobotRelativeSpeeds() {
  auto fl = m_frontLeft.GetState();
  auto fr = m_frontRight.GetState();
  auto bl = m_rearLeft.GetState();
  auto br = m_rearRight.GetState();
  return kDriveKinematics.ToChassisSpeeds(fl, fr, bl, br);
}
```

### Drive
Drive() is the swerve robot relative drive function it's the same function we use in DriveWithJoysticks but the boolean for fieldRelative is set to false
```cpp
void DriveSubsystem::Drive(units::meters_per_second_t xSpeed,
                           units::meters_per_second_t ySpeed,
                           units::radians_per_second_t rot,
                           bool fieldRelative) {
  
  auto states = kDriveKinematics.ToSwerveModuleStates(
      fieldRelative ? frc::ChassisSpeeds::FromFieldRelativeSpeeds(
                          xSpeed, ySpeed, rot, m_gyro.GetRotation2d())
                    : frc::ChassisSpeeds{xSpeed, ySpeed, rot});

  kDriveKinematics.DesaturateWheelSpeeds(&states, AutoConstants::kMaxSpeed);

  auto [fl, fr, bl, br] = states;

  m_frontLeft.SetDesiredState(fl);
  m_frontRight.SetDesiredState(fr); 
  m_rearLeft.SetDesiredState(bl);
  m_rearRight.SetDesiredState(br);

  FrontLeft = fl;
  FrontRight = fr;
  RearLeft =  bl;
  RearRight = br;
}
```

### Auto Constraints
Here we have the tranlation and rotation pids the maz speed the drive base radius in meters The last line is for what type of path replanning we wnat we just want the default so we don't do anything special for that
```cpp
HolonomicPathFollowerConfig( // HolonomicPathFollowerConfig, this should likely live in your Constants class
  PIDConstants(5.0, 0.0, 0.0), // Translation PID constants
  PIDConstants(5.0, 0.0, 0.0), // Rotation PID constants
  3.0_mps, // Max module speed, in m/s
  0.5_m, // Drive base radius in meters. Distance from robot center to furthest module.
  ReplanningConfig() // Default path replanning config. See the API for the options here
),
```

### Fieldflip
All it does it is a function that returns a boolean on whether or not the field should be flipped we will set this to a sendable chooser value there is a option to use the allince color that the program will auto read from the field but that is known to be unreliable

```cpp
bool DriveSubsystem::Fieldflip(){
  return m_fieldflip.GetSelected();
}
```

**Make sure all of this is in the DriveSubsystem because you are handing in ```this``` as the last parameter**

## Making auto commands 
Once you're done setting up the autobuilder you need to use that autobuilder

This is the basic code for getting a auto from the [pathplanner GUI](https://pathplanner.dev/pathplanner-gui.html) the function should be located in RobotContainer.cpp and called in the Robot.cpp AutonomousInit()

```cpp
frc2::CommandPtr RobotContainer::getAutonomousCommand(){
    return PathPlannerAuto("Auto").ToPtr();
}
```
All you need is the name of the Auto and Pathplanner will automatically make the command for you

### Sendable Chooser (String way)
We want the driver to be able to choose what auto to run the way we did it last year is a little strange but it works first you define a sendableChooser that will take in a String in RobotContainer.h
```cpp
  frc::SendableChooser<std::string> m_chooser;
```
Add options and send to SmartDashboard
```cpp
  m_chooser.SetDefaultOption("3.5 Note", "New Auto(4)");
  m_chooser.AddOption("3.5 Note", "New Auto(4)");
  m_chooser.AddOption("Side Note", "OneNoteSide");

  frc::SmartDashboard::PutData("Auto Modes", &m_chooser);
```
Lastly change the getAutonomousCommand Function to the following
```cpp
frc2::CommandPtr RobotContainer::getAutonomousCommand(){

    std::string autonomous = m_chooser.GetSelected();
    return PathPlannerAuto(autonomous).ToPtr();

}
```
m_chooser returns the string that the driver has selected and that string is handed into the PathPlannerAuto which gives us the desired auto
