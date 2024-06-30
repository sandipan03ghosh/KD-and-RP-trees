with open("fmnist-training.csv", "w") as file:
    for i in range(5):
        for j in range(5):
            file.write(str(i))
            if(not j==4):
                file.write(",")
        file.write("\n")
