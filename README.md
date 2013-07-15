lib_JSON_parser
===============

Flyport library for JSON parsing functionalities, released under GPL v.3.<br>
The library is a fork of http://sourceforge.net/projects/cjson/ from Dave Gamble: "an ultra-lightweight, portable, single-file, simple-as-can-be ANSI-C compliant JSON parser".<BR>
The library allowa to create and parse JSON strings, and read/write each parameter. An example of usage follows. More infos on wiki.openpicus.com.
<br>1) import files inside Flyport IDE using the external libs button.<br>
2) add following code example in FlyportTask.c:

<pre>
#include "taskFlyport.h"
#include "cJSON.h"
 
void FlyportTask()
{  
	vTaskDelay(20);
	/* reference JSON string */
	char *json_string = "{\
	 \"menu\": {\
		 \"id\": \"file\",\
		\"value\": \"File\",\
		\"popup\": {\
				\"menuitem\": 	[\
					{\"value\": \"New\", \"onclick\": \"CreateNewDoc()\"},\
					{\"value\": \"Open\", \"onclick\": \"OpenDoc()\"},\
					{\"value\": \"Close\", \"onclick\": \"CloseDoc()\"}\
						]\
			}\
		}\
	}";

	/* pointer to a root of JSON tree*/
	cJSON *json;
	json = cJSON_Parse(json_string );

	if (!json)  
	{
		char *error = (char*)cJSON_GetErrorPtr();
		UARTWrite(1,"\n\r An error was encountered\n\r");
		UARTWrite(1,error);
	}
	else
	{
		char *s_out = cJSON_Print(json);
		UARTWrite(1,"\r\nthe JSON  string received (formatted)\r\n");	
		UARTWrite(1,s_out);
		UARTWrite(1,"\r\n");

		/*to free the memory */
		free(s_out);	
		vTaskDelay(20);
		
		s_out = cJSON_PrintUnformatted(json);
		UARTWrite(1,"\r\nthe JSON  string received (unformatted)\r\n");	
		UARTWrite(1,s_out);
		UARTWrite(1,"\r\n");
		
		free(s_out);	
		vTaskDelay(20);
		
		cJSON *menu = cJSON_GetObjectItem(json,"menu");
		
		/* menu object has 3 children (id,value,popup)*/
		cJSON *id = cJSON_GetObjectItem(menu,"id");
		cJSON *value = cJSON_GetObjectItem(menu,"value");
		cJSON *popup = cJSON_GetObjectItem(menu,"popup");
		
		/* popup has  an array  (menuitem)*/
		 cJSON *array = cJSON_GetObjectItem(popup,"menuitem");
		
		/*size is the number of the array's items*/ 
		int size = cJSON_GetArraySize(array);
		
		/*get the item number 1*/
		cJSON *item1 = cJSON_GetArrayItem(array,1);
		s_out = cJSON_Print(item1);
		UARTWrite(1,"\r\nthe array's item number 1 is:\r\n");	
		UARTWrite(1,s_out);
		UARTWrite(1,"\r\n");
		free(s_out);
		vTaskDelay(20);
		
		/*delete the item number 2 from array*/
		cJSON_DeleteItemFromArray(array,2);
		
		/*get size = 2 now*/
		size = cJSON_GetArraySize(array);
		
		/*delete the JSON item called value  from JSON tree*/	
		cJSON_DeleteItemFromObject(json->child,"value");
		s_out = cJSON_Print(json);
		UARTWrite(1,"\r\nthe JSON string received without \"value\" and the array's item number 2 is:\r\n");	
		UARTWrite(1,s_out);
		UARTWrite(1,"\r\n");
		free(s_out);
		vTaskDelay(20);
		
		/*how to shift along JSON tree*/
		s_out = cJSON_Print(json->child->child->next);
		UARTWrite(1,"\r\nprint the JSON string from \"popup\" item (\"menuitem\" array will be printed)\r\n");	
		UARTWrite(1,s_out);
		UARTWrite(1,"\r\n");
		free(s_out);
		vTaskDelay(20);
		
		/* let's see how to create a JSON*/

		/* JSON to be created
		{
		"menu": 
			{
				"header": "JSON example",
				"items": [
						{"id": "Open"},
						{"id": "OpenNew", "label": "JSON"},
				]
			}
		}

		*/
		/*create a JSON Object*/
		
		cJSON *root = cJSON_CreateObject();
		cJSON *menu_1 = cJSON_CreateObject();
		
		/*add  objects to the JSON */
		cJSON_AddItemToObject(root,"menu",menu_1);
		cJSON_AddItemToObject(menu_1,"header",cJSON_CreateString("JSON example"));
		cJSON *array_1 = cJSON_CreateArray();
		
		/*add an array  to the JSON */
		cJSON_AddItemToObject(menu_1,"items",array_1);

		cJSON *item_1 = cJSON_CreateObject();
		cJSON *item_2= cJSON_CreateObject();

		cJSON_AddItemToObject(item_1,"id",cJSON_CreateString("Open"));

		cJSON_AddItemToObject(item_2,"id",cJSON_CreateString("OpenNew"));
		cJSON_AddItemToObject(item_2,"label",cJSON_CreateString("JSON"));

		/*attach elements  to the array*/
		cJSON_AddItemToArray(array_1,item_1);
		cJSON_AddItemToArray(array_1,item_2 );

		/*print the JSON  created*/
		s_out = cJSON_Print(root);
		UARTWrite(1,"\r\nthe JSON created\r\n");	
		UARTWrite(1,s_out);
		UARTWrite(1,"\r\n");
		
		/*free the memory allocated*/
		free(s_out);
		vTaskDelay(20);
		
		/*Detach the array from JSON leaving it in append variable*/
		cJSON *append = cJSON_DetachItemFromObject(menu_1,"items");
		s_out = cJSON_Print(root);
		UARTWrite(1,"\r\nthe JSON  without the array\r\n");	
		UARTWrite(1,s_out);
		UARTWrite(1,"\r\n");
		free(s_out);
		vTaskDelay(20);
		
		/* create a string type json*/
		cJSON *json_string = cJSON_CreateString("string");
		cJSON_AddItemToObject(menu_1,"name",json_string);
		/* create a number type json*/
		cJSON * json_numb = cJSON_CreateNumber(1.345);
		cJSON_AddItemToObject(menu_1,"number",json_numb);
		/* create an integer array*/
		int number[10] = {0,1,2,3,4,5,6,7};
		cJSON * int_array = cJSON_CreateIntArray(number,10);
		cJSON_AddItemToObject(menu_1,"int_array",int_array);
		s_out = cJSON_Print(root);
		UARTWrite(1,"\r\nthe JSON with a string type item,number type item and an integer type array\r\n");	
		UARTWrite(1,s_out);
		UARTWrite(1,"\r\n");
		free(s_out);
		vTaskDelay(20);
	 
		/*the last print should appear as

		{
		"menu":	{
			"header":	"JSON example",
			"name":	"string",
			"number":	1.345000,
			"int_array":	[0, 1, 2, 3, 4, 5, 6, 7, 0, 0]
			}
		}
		*/

	}

	while(1);
}

</pre>