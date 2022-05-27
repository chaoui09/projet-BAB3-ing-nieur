#code pour les classes de Json (numéro de la tâche ?)
# projet-BAB3-ing-nieur
import json
import copy
import os
import numpy
import statistics

class MotionFeatures:
    ''' 
    Class that regroups set of functions that compute and return local coordinates and velocities from 1 json folder.
    Each function returns the features along one or subset of joints.
    '''
    
    def __init__(self,valdemedium,joint=0):
        # self.jsons_folder_path = open(nadjajson,'r')
        self.jsons_folder_path = valdemedium
        self.results = []
        self.count_list =[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0]
        self.dict_pose_keypoints_2d ={}
        indice_list=[]
        for root, subdirs, files in os.walk(valdemedium):
            for file in files:
                if file.endswith(".json"):
                    with open(os.path.join(root,file), "r") as f:
                        data = json.load(f)
                        for state in data['people']:
                            valeur_key = 1
                            for i in range(0,len(state['face_keypoints_2d']),3):
                                self.dict_pose_keypoints_2d[valeur_key] =(state['face_keypoints_2d'][i], state['face_keypoints_2d'][i+1],state['face_keypoints_2d'][i+2])
                                valeur_key +=1 
                                if state['face_keypoints_2d'][i] == 0:
                                    self.count_list[valeur_key-2]+=1
                            self.results.append(copy.deepcopy(self.dict_pose_keypoints_2d))
        counter=0                     
        for count in self.count_list:
            if count > 10:
                indice_list.append(counter)
            counter +=1  
        for dictfinal in self.results:
            for indice in indice_list:
                del dictfinal[indice+1]         
        self.listde1a59=[i for i in range(1,69+1)]     
        '''
        Read all json files inside the folder and get global coordinate data
        '''    
    def get_local_coord_of_joint(self, joint,frames):
        return self.get_local_coord(joint,frames) #return local coordinate for joint 

    def get_local_velocity_of_joint(self, joint, frames):
        return self.get_local_velocity(joint, frames) #return local velocity for joint

    def get_local_moy_of_joints(self,frames):
        return self.get_local_moy(joint)
        
    def get_local_coord(self,joint,frames):
        self.dict_finalevrai={}
        key=1
        listframe=[i for i in range(1,2+1)]
        self.resultfinal=[]
        # [nframes, joint, 3(xyz)] => pos_local = pos_global - pos_global[:,0]
        for frames in listframe:
            for joint in self.listde1a59:
                pos_global = tuple(numpy.subtract(self.results[frames][joint], self.results[frames][33]))
                self.dict_finalevrai[joint]=(pos_global)   
            self.resultfinal.append(copy.deepcopy(self.dict_finalevrai))            
        return self.resultfinal
        '''
        Global coordinates are cartesian xyz position in the world space coordinate
        Local coordinates are cartesian xyz position in the local space where the origin is the root joint (chosen a the 'Hip' joint) 
        Considering pos_global, the global cartesian coordinate shaped as [nframes, joint, 3(xyz)] => pos_local = pos_global - pos_global[:,0] if 0 is the index of root joint 
        '''
        pass

    def get_local_velocity(self, joints,frames):
        self.resultvelocity=[]
        lapped=0
        self.dict_finalevelocity={}
        listframes=[j for j in range(1,62+1)]#62 correspond to the number of frames to analysis
        for frames in listframes:
            for joints in self.listde1a59:
                velocity = tuple(numpy.subtract(self.resultfinal[frames+1][joints], self.resultfinal[frames][joints]))
                velocity2=[abs(ele)for ele in velocity]
                self.resultvelocity.append(velocity2)
           
        self.res=[sum([abs(ele) for ele in sub])/62 for sub in self.resultvelocity]  #average speed of each joint in absolute value
        return self.res
        print(numpy.std(self.res))#total standard deviation of the mean
        '''
        Local velocities are differences of local positions between two frames
        Considering pos_local shaped as [nframes, 3(xyz)] => v_local = pos_local[f+1] - pos_local[f] 
        '''       
if __name__ == '__main__':
    valdemedium = 'C:\\Users\\fcogl\\Downloads\\valdejson\\valdemedium' #path/to/data
    mf1 = MotionFeatures(valdemedium)
    mf1.get_local_coord(1,1)  
    print(mf1.get_local_velocity(1,1))
