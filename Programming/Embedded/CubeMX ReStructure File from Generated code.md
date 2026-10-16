Related to [[Embedded]], [[CubeMX]]
#restructure_CubeMX_gen 

If you are looking to throw the generated code that CubeMX gives you such as the Src/ directory and the Inc/ you need to nvim/edit the .ioc file and the go manually change the value of 
`ProjectManager.MainLocation=Src` to `ProjectManager.MainLocation=Core/Src`

that will update the ioc. 
- Updating the ioc is dangerous and should not be done for other complex tasks like updating registers or even other file management tasks besides simple ones like this where you just change the file structure