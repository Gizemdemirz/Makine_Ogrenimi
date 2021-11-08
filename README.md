# Makine_Ogrenimi

import numpy as np 
import pandas as pd 
from subprocess import check_output
print(check_output(["ls", "../input"]).decode("utf8"))
import matplotlib.pyplot as plt
import matplotlib.animation as animation

data = pd.read_csv('../input/house-prices-advanced-regression-techniques/train.csv')

x = data['GrLivArea']
y = data['SalePrice']

x = (x - x.mean()) / x.std()
x = np.c_[np.ones(x.shape[0]), x] 

alpha = 0.01 
iterations = 2000 
m = y.size 
np.random.seed(123) 
theta = np.random.rand(2) 



def gradient_descent(x, y, theta, iterations, alpha):
    past_costs = []
    past_thetas = [theta]
    for i in range(iterations):
        prediction = np.dot(x, theta)
        error = prediction - y
        cost = 1/(2*m) * np.dot(error.T, error)
        past_costs.append(cost)
        theta = theta - (alpha * (1/m) * np.dot(x.T, error))
        past_thetas.append(theta)
        
    return past_thetas, past_costs


past_thetas, past_costs = gradient_descent(x, y, theta, iterations, alpha)
theta = past_thetas[-1]


print("Gradient Descent: {:.2f}, {:.2f}".format(theta[0], theta[1]))

plt.title('Maliyet Fonksiyonu')
plt.xlabel('İterasyon Sayısı')
plt.ylabel('Maliyet')
plt.plot(past_costs)
plt.show()


fig = plt.figure()
ax = plt.axes()
plt.title('Satış Fiyatı - Yaşam Alanı')
plt.xlabel('Fit kare cinsinden Yaşam Alanı (normalleştirilmiş)')
plt.ylabel('Satış Fiyatı ($)')
plt.scatter(x[:,1], y, color='red')
line, = ax.plot([], [], lw=2)
annotation = ax.text(-1, 700000, '')
annotation.set_animated(True)
plt.close()


def init():
    line.set_data([], [])
    annotation.set_text('')
    return line, annotation


def animate(i):
    x = np.linspace(-5, 20, 1000)
    y = past_thetas[i][1]*x + past_thetas[i][0]
    line.set_data(x, y)
    annotation.set_text('Maliyet = %.2f e10' % (past_costs[i]/10000000000))
    return line, annotation

anim = animation.FuncAnimation(fig, animate, init_func=init,
                               frames=300, interval=0, blit=True)

anim.save('animation.gif', writer='imagemagick', fps = 30)

import io
import base64
from IPython.display import HTML

filename = 'animation.gif'

video = io.open(filename, 'r+b').read()
encoded = base64.b64encode(video)
HTML(data='''<img src="data:image/gif;base64,{0}" type="gif" />'''.format(encoded.decode('ascii')))
