# Recipe_Project
In this project we can search recipe for given ingredients.
import requests
import json
import pandas as pd
import warnings

warnings.filterwarnings("ignore")
pd.set_option('display.max_rows', None)
pd.set_option('display.max_columns', None)
pd.set_option('display.width', None)
pd.set_option('display.max_colwidth', -1)


def api_response(ingredient, app_id, app_key, number_recipe):
    response = requests.get('https://api.edamam.com/search?q={}&app_id={}&app_key={}&to={}'.format(ingredient, app_id, app_key, number_recipe))
    data = response.json()

    with open('resp_text.txt', 'w') as file:
        json.dump(data, file,indent=4)

    return data['hits']

def nutrition_api_response(nutrition_app_id, nutrition_app_key, input_data):
    url = "https://api.edamam.com/api/nutrition-details?app_id={}&app_key={}".format(nutrition_app_id, nutrition_app_key)

    nutrition_responses = requests.post(url, json=input_data)
    nutrition_res = nutrition_responses.json()

    print(nutrition_res)


def recipes_search():
    ingredient = input('Enter an ingredient that you want to recipe for: ')
    number_recipe = int(input('Enter How many recipe result you want?: '))

    if number_recipe > 0:
        sort_recipe = ""
        app_id = 'fcd86dec'
        app_key = 'd65c9e7f5776230918de24b756de831f'
        responses = api_response(ingredient, app_id, app_key, number_recipe)


        proxy_types = {"label": "1", "calories": "2", "totalWeight": "3", "totalTime": "4"}
        while True:
            option = input("""Choose and Type option whatever you want to short the recipe )
            option 1  - label
            option 2  - calories
            option 3  - totalWeight
            option 4  - totalTime
            choose one option from above > """)
            if option not in proxy_types:
                print("Invalid option selected. Choose again.")
                continue
            sort_recipe = option
            break

        cnt = 0
        df = pd.DataFrame({'Sorted Data': ['Dummy'], 'Value': ['Dummy_Values']})
        df_recipe = pd.DataFrame({'Recipe_Name': ['Dummy'], 'Recipe_Value': ['Dummy_Values']})
        df_calories = pd.DataFrame({'Recipe_CName': ['Dummy'], 'Recipe_CValue': ['Dummy_Values']})
        for result in responses:
            recipe = result['recipe']
            if sort_recipe != "no option":

                key = recipe[sort_recipe]

                recipe_label = recipe['label']
                uri = recipe['uri']
                source = recipe['source']
                url = recipe['url']
                recipe_calories = recipe['calories']
                total_time = recipe['totalTime']
                ingredient = recipe['ingredientLines']

                print("Recipe Name : ", recipe_label)
                print("Recipe source Name : ", source)
                print("Recipe source URL : ", url)
                print("Recipe URI :", uri)
                print("Calories for recipe : ", recipe_calories)
                print("Total Time : ", total_time)
                print("Ingredient List : ", recipe['ingredientLines'])

                tuples_list = [recipe_label, uri, source, url, recipe_calories, total_time, ingredient]
                cnt = cnt + 1
            else:
                print("Recipe Name : ", recipe['label'])
                print("Recipe URI :", recipe['uri'])
                print("Recipe source Name : ", recipe['source'])
                print("Recipe source URL : ", recipe['url'])
                print("Calories for each recipe : ", recipe['calories'])
                print("Total Time : ", recipe['totalTime'])
                print("Ingredient List : ")
                for ingredient in recipe['ingredientLines']:
                    print(ingredient, sep='\'', end='\n')
                cnt = cnt + 1
                print()
            df = df.append(pd.DataFrame({'Sorted Data': [key], 'Value': [tuples_list]}))
            df_recipe = df_recipe.append(pd.DataFrame({'Recipe_Name': [recipe_label], 'Recipe_Value': [tuples_list]}))
            df_calories = df_calories.append(pd.DataFrame({'Recipe_CName': [recipe_calories], 'Recipe_CValue': [tuples_list]}))

        df1 = df.iloc[1:].sort_values('Sorted Data')
        print("Total no of recipe found is: {}".format(cnt))
        print("SORTED RECIPE LIST : ")
        print(df1)
        print()
        print()

        max_cal = float(input('Enter the Max Calorie range : '))
        min_cal = float(input('Enter the Min Calorie range  : '))
        print("Recipe Result Given Calorie Range: ")
        df_calories = df_calories.iloc[1:]
        # print(df_calories['Recipe_CName'])
        df_calories = df_calories[df_calories['Recipe_CName'].between(min_cal, max_cal,inclusive = True)]
        print(df_calories)

        print("Menu List: ")
        df_recipe = df_recipe.iloc[1:]
        print(df_recipe['Recipe_Name'])
        print()
        ask_recipe = input('Enter the recipe you want to choose?  : ')
        print("User Selected Recipe Details: ")
        for recipe1 in df_recipe['Recipe_Name']:
            if recipe1 == ask_recipe:
                print(df_recipe.loc[df_recipe['Recipe_Name'] == recipe1])
                break

        nutrition_app_id = '06a1fd21'
        nutrition_app_key = 'e88e97f80d14a8a12a1ca448842c1fa6'
        data = {}
        data["title"] = ask_recipe
        for recipe in responses:
            if recipe['recipe']['label'] == ask_recipe:
                data["ingr"] = recipe['recipe']['ingredientLines']
                # print(data["ingr"])
        #print(data)
        print()
        print("Recipe Related Nutrition Details: ")
        nutrition_api_response(nutrition_app_id, nutrition_app_key, data)

    else:
        print("Wrong entry,please Enter valid number:")
        recipes_search()

recipes_search()
